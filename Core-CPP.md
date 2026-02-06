# GhostWire Audio Core - Complete Line-by-Line Code Explanation (Part 2)

This continues the detailed explanation of the GhostWire Audio Core library.

## Table of Contents

1. [Source Files (Implementation)](#source-files-implementation)
2. [Test Files](#test-files)
3. [Example Programs](#example-programs)
4. [Summary & Key Concepts](#summary--key-concepts)

---

## Source Files (Implementation)

Source files (`.cpp`) contain the actual implementation of functions declared in headers.

### src/version.cpp

```cpp
#include "gw/core/version.h"

namespace gw::core {
    const char *get_version_string() {
        return "GhostWire Core v0.1.0";
    }
}
```

**Line-by-line:**
- **Line 1**: Include the corresponding header (quotes mean "project file")
- **Line 3**: Open the namespace (must match header exactly)
- **Line 4**: Function definition - implements what was declared in header
- **Line 5**: `return "..."` - String literal returns pointer to const char array
- **Line 6**: Close function body
- **Line 7**: Close namespace

---

### src/audio_format.cpp

```cpp
#include "gw/core/audio_format.h"

namespace gw::core {
    AudioFormat::AudioFormat(uint32_t sample_rate,
                             uint32_t num_channels,
                             uint32_t bit_depth)
        : sample_rate_(sample_rate),
          num_channels_(num_channels),
          bit_depth_(bit_depth) {
    }
```

**Member Initializer List (Lines 7-9):**
- `:` starts the member initializer list
- `sample_rate_(sample_rate)` - Initialize member `sample_rate_` with parameter `sample_rate`
- `,` separates each initializer
- This is more efficient than assigning in the function body
- Function body is empty `{}` - all work done in initializer list

```cpp
    size_t AudioFormat::get_bytes_per_sample() const {
        // Integer division rounds down, so we round up
        return (bit_depth_ + 7) / 8;
    }
```

**Rounding Up Division:**
- Example: 24 bits → (24 + 7) / 8 = 31 / 8 = 3 bytes ✓
- Example: 16 bits → (16 + 7) / 8 = 23 / 8 = 2 bytes ✓
- Example: 17 bits → (17 + 7) / 8 = 24 / 8 = 3 bytes ✓ (rounds up!)
- Adding 7 before dividing ensures we round up

```cpp
    size_t AudioFormat::get_bytes_per_frame() const {
        return get_bytes_per_sample() * num_channels_;
    }
```

- A "frame" is one sample for all channels simultaneously
- Stereo 24-bit: 3 bytes/sample × 2 channels = 6 bytes/frame

```cpp
    bool AudioFormat::operator==(const AudioFormat &other) const {
        return sample_rate_ == other.sample_rate_ &&
               num_channels_ == other.num_channels_ &&
               bit_depth_ == other.bit_depth_;
    }
```

**Operator Overloading:**
- Allows writing `format1 == format2`
- `&&` - Logical AND (all three must be true)
- `other.sample_rate_` - Access another object's private member (allowed within class)

```cpp
    bool AudioFormat::operator!=(const AudioFormat &other) const {
        return !(*this == other);
    }
}
```

**Reusing == operator:**
- `*this` - Dereference the `this` pointer to get current object
- `!(...)`  - Logical NOT (opposite of ==)
- DRY principle: Don't Repeat Yourself

---

### src/audio_buffer.cpp

```cpp
#include "gw/core/audio_buffer.h"
#include <cstdlib>      // for aligned_alloc, free
#include <cstring>      // for memcpy, memset
#include <algorithm>    // for std::min

namespace gw::core {
    static constexpr size_t ALIGNMENT = 32;
```

**Why 32-byte alignment?**
- AVX instructions (SIMD) require 32-byte alignment
- SIMD = Single Instruction Multiple Data
- Process 8 floats (32 bytes) simultaneously
- Massive performance improvement for DSP

```cpp
    AudioBuffer::AudioBuffer(size_t num_channels, size_t num_samples)
        : num_channels_(num_channels),
          num_samples_(num_samples),
          channel_data_(nullptr) {
        
        if (num_channels_ == 0 || num_samples_ == 0) {
            return; // Empty buffer
        }
        
        channel_data_ = new float *[num_channels_];
```

**Two-level allocation:**
1. Allocate array of pointers: `float *[num_channels_]`
2. Each pointer will point to one channel's data (allocated next)

```cpp
        for (size_t ch = 0; ch < num_channels_; ++ch) {
            const size_t bytes = num_samples_ * sizeof(float);
            const size_t aligned_bytes = ((bytes + ALIGNMENT - 1) / ALIGNMENT) * ALIGNMENT;
            
            channel_data_[ch] = static_cast<float *>(std::aligned_alloc(ALIGNMENT, aligned_bytes));
            
            if (channel_data_[ch]) {
                std::memset(channel_data_[ch], 0, bytes);
            }
        }
    }
```

**Alignment Calculation:**
- Formula: `((bytes + ALIGNMENT - 1) / ALIGNMENT) * ALIGNMENT`
- This rounds up to the next multiple of ALIGNMENT
- Example: 100 bytes with 32-byte alignment
  - `(100 + 32 - 1) / 32 = 131 / 32 = 4` (integer division)
  - `4 * 32 = 128` (rounded up!)

**CRITICAL:** Memory allocated with `aligned_alloc()` MUST be freed with `std::free()`, NOT `delete`!

```cpp
    AudioBuffer::~AudioBuffer() {
        free_memory();
    }
```

**Destructor** - Called automatically when object destroyed

```cpp
    AudioBuffer::AudioBuffer(AudioBuffer &&other) noexcept
        : num_channels_(other.num_channels_),
          num_samples_(other.num_samples_),
          channel_data_(other.channel_data_) {
        
        other.num_channels_ = 0;
        other.num_samples_ = 0;
        other.channel_data_ = nullptr;
    }
```

**Move Constructor:**
1. Copy other's data to this
2. Null out other's pointers
3. **Why?** Prevents other's destructor from freeing our memory
4. "Stealing" resources instead of copying

```cpp
    void AudioBuffer::copy_from(const AudioBuffer &source) {
        if (!channel_data_ || !source.channel_data_) return;
        
        const size_t channels_to_copy = std::min(num_channels_, source.num_channels_);
        const size_t samples_to_copy = std::min(num_samples_, source.num_samples_);
        
        for (size_t ch = 0; ch < channels_to_copy; ++ch) {
            std::memcpy(channel_data_[ch],
                        source.channel_data_[ch],
                        samples_to_copy * sizeof(float));
        }
    }
```

**Safe Copying:**
- Use `std::min()` to avoid buffer overflow
- `memcpy()` - Fast memory copy (optimized by compiler)
- Copies minimum dimensions only

```cpp
    void AudioBuffer::free_memory() {
        if (channel_data_) {
            for (size_t ch = 0; ch < num_channels_; ++ch) {
                std::free(channel_data_[ch]);  // FREE, not delete!
            }
            delete[] channel_data_;  // But this uses delete[]
            channel_data_ = nullptr;
        }
    }
}
```

**Memory Management Rules:**
- `aligned_alloc()` → `std::free()`
- `new T[n]` → `delete[]`
- `new T` → `delete`
- Mixing these causes undefined behavior (crashes, corruption)

---

### src/buffer_view.cpp

```cpp
#include "gw/core/buffer_view.h"
#include "gw/core/audio_buffer.h"
#include <cstring>
#include <algorithm>

namespace gw::core {
    BufferView::BufferView()
        : data_(nullptr),
          num_samples_(0) {
    }
```

**Default constructor** - Creates empty view

```cpp
    BufferView::BufferView(float *data, size_t num_samples)
        : data_(data),
          num_samples_(num_samples) {
    }
```

**From pointer constructor** - Wraps existing data

```cpp
    BufferView::BufferView(AudioBuffer &buffer, size_t channel)
        : data_(buffer.get_channel_data(channel)),
          num_samples_(buffer.get_num_samples()) {
    }
```

**From AudioBuffer constructor** - Creates view of one channel

```cpp
    BufferView BufferView::subview(size_t offset, size_t count) const {
        if (offset >= num_samples_ || !data_) {
            return {}; // Empty view
        }
        
        const size_t remaining = num_samples_ - offset;
        const size_t actual_count = (count == 0) ? remaining : std::min(count, remaining);
        
        return {data_ + offset, actual_count};
    }
```

**Subview Creation:**
- `return {};` - Aggregate initialization of empty BufferView
- `data_ + offset` - Pointer arithmetic (advance pointer)
- **Ternary operator:** `condition ? if_true : if_false`
  - If count is 0, use all remaining samples
  - Otherwise, use minimum of requested and available

```cpp
    void BufferView::fill(float value) {
        if (!data_) return;
        
        for (size_t i = 0; i < num_samples_; ++i) {
            data_[i] = value;
        }
    }
```

**Fill** - Simple loop, could be optimized with SIMD

```cpp
    void BufferView::clear() {
        if (!data_) return;
        std::memset(data_, 0, num_samples_ * sizeof(float));
    }
}
```

**Clear** - `memset` is highly optimized (often uses SIMD internally)

---

### src/ring_buffer.cpp

```cpp
#include "gw/core/ring_buffer.h"
#include <cstdlib>
#include <cstring>
#include <algorithm>

namespace gw::core {
    RingBuffer::RingBuffer(const size_t capacity)
        : capacity_(capacity + 1),
          buffer_(nullptr),
          write_pos_(0),
          read_pos_(0) {
```

**Why capacity + 1?**
- Problem: If `read_pos_ == write_pos_`, buffer could be empty OR full
- Solution: Waste one slot
- Empty: `read_pos_ == write_pos_`
- Full: `(write_pos_ + 1) % capacity_ == read_pos_`

```cpp
        if (capacity_ > 0) {
            buffer_ = new float[capacity_];
            std::memset(buffer_, 0, capacity_ * sizeof(float));
        }
    }
```

**Simple allocation** - Regular `new[]` (not aligned like AudioBuffer)

```cpp
    size_t RingBuffer::write(const float *data, size_t count) {
        if (!buffer_ || !data) return 0;
        
        const size_t available = get_available_write();
        const size_t to_write = std::min(count, available);
        
        size_t write_idx = write_pos_.load(std::memory_order_relaxed);
```

**Atomic Load:**
- `write_pos_` is `std::atomic<size_t>`
- `.load()` reads the atomic value
- `std::memory_order_relaxed` - No synchronization (we're the only writer)

```cpp
        for (size_t i = 0; i < to_write; ++i) {
            buffer_[write_idx] = data[i];
            write_idx = (write_idx + 1) % capacity_;
        }
```

**Wraparound with Modulo:**
- `%` - Modulo operator (remainder after division)
- Example with capacity_ = 8:
  - write_idx = 7: `(7 + 1) % 8 = 0` (wraps to beginning!)
  - write_idx = 3: `(3 + 1) % 8 = 4` (normal increment)

```cpp
        write_pos_.store(write_idx, std::memory_order_release);
        return to_write;
    }
```

**Memory Ordering - Release:**
- All previous memory writes visible to threads that acquire
- Ensures reader sees data before seeing updated write_pos_
- Critical for correctness!

```cpp
    size_t RingBuffer::read(float *data, size_t count) {
        if (!buffer_ || !data) return 0;
        
        const size_t available = get_available_read();
        const size_t to_read = std::min(count, available);
        
        size_t read_idx = read_pos_.load(std::memory_order_relaxed);
        for (size_t i = 0; i < to_read; ++i) {
            data[i] = buffer_[read_idx];
            read_idx = (read_idx + 1) % capacity_;
        }
        
        read_pos_.store(read_idx, std::memory_order_release);
        return to_read;
    }
```

**Read is mirror of Write** - Same pattern, opposite direction

```cpp
    size_t RingBuffer::get_available_read() const {
        const size_t write_idx = write_pos_.load(std::memory_order_acquire);
        const size_t read_idx = read_pos_.load(std::memory_order_acquire);
```

**Memory Ordering - Acquire:**
- See all writes before corresponding release
- Ensures we see writer's data

```cpp
        if (write_idx >= read_idx) {
            return write_idx - read_idx;
        } else {
            return capacity_ - read_idx + write_idx;
        }
    }
```

**Two Cases:**

**Case 1:** No wraparound
```
[___R____W___]
     ^^^^^ = write - read
```

**Case 2:** Wraparound
```
[__W_____R___]
  ^^     ^^^ = (capacity - read) + write
```

```cpp
    size_t RingBuffer::get_available_write() const {
        const size_t available_read = get_available_read();
        return (capacity_ - 1) - available_read;
    }
```

**Available Write:**
- Total usable space: `capacity_ - 1` (remember the wasted slot?)
- Minus what's already used: `available_read`

```cpp
    void RingBuffer::clear() {
        write_pos_.store(0, std::memory_order_relaxed);
        read_pos_.store(0, std::memory_order_relaxed);
    }
```

**Clear** - Just reset positions (data stays but will be overwritten)
**WARNING:** NOT thread-safe during use! Only call when audio stopped.

---

## Test Files

### tests/test_main.cpp

```cpp
#include <iostream>

void test_audio_format();
void test_audio_buffer();
void test_buffer_view();
void test_ring_buffer();

int main() {
    std::cout << "=== Running GhostWire Core Tests ===" << std::endl;
    
    try {
        std::cout << "\n[1/4] Testing AudioFormat..." << std::endl;
        test_audio_format();
        std::cout << "  ✓ AudioFormat tests passed" << std::endl;
        
        // ... similar for other tests ...
        
        std::cout << "\n=== All tests passed! ===" << std::endl;
        return 0;
    } catch (const std::exception &e) {
        std::cerr << "\n✗ Test failed: " << e.what() << std::endl;
        return 1;
    }
}
```

**Try-Catch Block:**
- `try { ... }` - Run code that might throw exceptions
- `catch (const std::exception &e)` - Catch any standard exception
- `e.what()` - Get error message
- Return 0 = success, non-zero = failure (Unix convention)

---

### tests/test_audio_format.cpp

```cpp
#include <gw/core/audio_format.h>
#include <cassert>
#include <iostream>

void test_audio_format() {
    const gw::core::AudioFormat format(48000, 2, 24);
    
    assert(format.get_sample_rate() == 48000);
    assert(format.get_num_channels() == 2);
    assert(format.get_bit_depth() == 24);
```

**Assertions:**
- `assert(condition)` - If false, program crashes with error
- Used for testing - immediate failure if something wrong
- Only enabled in debug builds (disabled in release for performance)

```cpp
    assert(format.get_bytes_per_sample() == 3); // 24 bits = 3 bytes
    assert(format.get_bytes_per_frame() == 6); // 2 channels * 3 bytes
```

**Calculation Verification:**
- 24 bits / 8 bits/byte = 3 bytes
- 2 channels × 3 bytes/sample = 6 bytes/frame

```cpp
    const gw::core::AudioFormat format2(48000, 2, 24);
    assert(format == format2);
    
    const gw::core::AudioFormat format3(44100, 2, 24);
    assert(format != format3);
    
    std::cout << "  - Construction: OK" << std::endl;
    std::cout << "  - Byte calculations: OK" << std::endl;
    std::cout << "  - Equality: OK" << std::endl;
}
```

**Operator Testing:**
- Identical formats should be equal
- Different formats should not be equal

---

### tests/test_audio_buffer.cpp

```cpp
void test_audio_buffer() {
    gw::core::AudioBuffer buffer(2, 512);
    
    // Test zero-initialization
    for (size_t ch = 0; ch < 2; ++ch) {
        const float *data = buffer.get_channel_data(ch);
        assert(data != nullptr);
        for (size_t i = 0; i < 512; ++i) {
            assert(data[i] == 0.0f);
        }
    }
```

**Nested Loops:**
- Outer loop: iterate channels
- Inner loop: iterate samples
- Verify every single sample is 0.0

```cpp
    // Test sample access
    buffer.set_sample(0, 100, 0.5f);
    assert(buffer.get_sample(0, 100) == 0.5f);
    assert(buffer.get_sample(1, 100) == 0.0f);
```

**Write and Verify:**
- Set channel 0, sample 100 to 0.5
- Verify it was set
- Verify other channel unchanged

```cpp
    // Test copy_from
    gw::core::AudioBuffer source(2, 512);
    for (size_t i = 0; i < 512; ++i) {
        float phase = static_cast<float>(i) / 512.0f;
        source.set_sample(0, i, std::sin(2.0f * 3.14159f * phase));
    }
    
    buffer.copy_from(source);
    
    for (size_t i = 0; i < 512; ++i) {
        assert(buffer.get_sample(0, i) == source.get_sample(0, i));
    }
```

**Sine Wave Generation:**
- `phase` goes from 0.0 to 1.0
- `2π * phase` goes from 0 to 2π (full circle)
- `sin(2π * phase)` generates one complete sine wave cycle

```cpp
    // Test move semantics
    const gw::core::AudioBuffer moved = std::move(buffer);
    assert(moved.get_num_channels() == 2);
    assert(moved.get_num_samples() == 512);
    assert(buffer.get_channel_data(0) == nullptr);
```

**Move Testing:**
- `std::move(buffer)` - Cast to rvalue reference
- Forces move constructor
- Original should be empty (moved-from state)
- Moved-to object should have all the data

---

### tests/test_buffer_view.cpp

```cpp
void test_buffer_view() {
    gw::core::AudioBuffer buffer(2, 100);
    
    for (size_t i = 0; i < 100; ++i) {
        buffer.set_sample(0, i, static_cast<float>(i));
    }
    
    gw::core::BufferView view(buffer, 0);
    
    assert(view.size() == 100);
    assert(!view.empty());
    assert(view[0] == 0.0f);
    assert(view[50] == 50.0f);
```

**Subscript Operator Testing:**
- `view[i]` uses `operator[]`
- Should match buffer values

```cpp
    auto subview = view.subview(10, 20);
    assert(subview.size() == 20);
    assert(subview[0] == 10.0f);
    assert(subview[19] == 29.0f);
```

**Subview Indexing:**
- Subview starts at offset 10 in original
- `subview[0]` = original `view[10]` = 10.0
- `subview[19]` = original `view[29]` = 29.0

```cpp
    subview.fill(99.0f);
    
    assert(buffer.get_sample(0, 10) == 99.0f);
    assert(buffer.get_sample(0, 29) == 99.0f);
    assert(buffer.get_sample(0, 9) == 9.0f);
    assert(buffer.get_sample(0, 30) == 30.0f);
```

**View Modification:**
- Fill subview (indices 10-29)
- Verify original buffer modified
- Verify surrounding values unchanged
- **This proves the view actually points to the buffer!**

---

### tests/test_ring_buffer.cpp

```cpp
void test_ring_buffer() {
    gw::core::RingBuffer ring(100);
    
    assert(ring.get_capacity() == 101);
    assert(ring.get_available_read() == 0);
    assert(ring.get_available_write() == 100);
```

**Initial State:**
- Capacity is 101 (100 + 1 for sentinel)
- Nothing to read yet
- Can write 100 samples

```cpp
    const std::vector<float> write_data = {1.0f, 2.0f, 3.0f, 4.0f, 5.0f};
    const size_t written = ring.write(write_data.data(), write_data.size());
    
    assert(written == 5);
    assert(ring.get_available_read() == 5);
    assert(ring.get_available_write() == 95);
```

**Write Test:**
- `std::vector` - Dynamic array from STL
- `.data()` - Get pointer to internal array
- `.size()` - Get number of elements
- After writing 5, available read = 5, available write = 95

```cpp
    std::vector<float> read_data(5);
    const size_t read_count = ring.read(read_data.data(), read_data.size());
    
    assert(read_count == 5);
    assert(ring.get_available_read() == 0);
    
    for (size_t i = 0; i < 5; i++) {
        assert(read_data[i] == write_data[i]);
    }
```

**Read Test:**
- Read back what we wrote
- Verify data matches
- Buffer now empty

```cpp
    const std::vector<float> large_data(60, 7.0f);
    ring.write(large_data.data(), large_data.size());
    ring.write(large_data.data(), large_data.size());
    
    assert(ring.get_available_read() == 100);
```

**Wraparound Test:**
- `std::vector<float>(60, 7.0f)` - 60 elements, all 7.0
- Write 60, then write 60 more
- Only 100 fit (40 from second write discarded)
- Write position wrapped around to beginning

---

## Example Programs

### examples/hello_ghostwire.cpp

```cpp
#include <gw/core/version.h>
#include <gw/core/audio_format.h>
#include <gw/core/audio_buffer.h>
#include <gw/core/buffer_view.h>
#include <gw/core/ring_buffer.h>
#include <iostream>
#include <cmath>

int main() {
    std::cout << "=== GhostWire Audio Engine ===" << std::endl;
    std::cout << gw::core::get_version_string() << std::endl;
```

**Demonstration Program:**
- Shows how to use each component
- Not a test - just examples

```cpp
    gw::core::AudioFormat format(48000, 2, 24);
    std::cout << "  Sample Rate: " << format.get_sample_rate() << " Hz" << std::endl;
    std::cout << "  Channels: " << format.get_num_channels() << std::endl;
    std::cout << "  Bit Depth: " << format.get_bit_depth() << " bits" << std::endl;
```

**AudioFormat Demo:**
- Create 48kHz stereo 24-bit format
- Print all properties

```cpp
    gw::core::AudioBuffer buffer(2, 512);
    
    float frequency = 440.0f;
    float sample_rate = 48000.0f;
    for (size_t i = 0; i < buffer.get_num_samples(); i++) {
        float sample = std::sin(2.0f * 3.14159f * frequency * static_cast<float>(i) / sample_rate);
        buffer.set_sample(0, i, sample);
    }
```

**440 Hz Sine Wave:**
- A440 = Musical note A above middle C
- `i / sample_rate` = time in seconds
- `2π * frequency * time` = angle
- `sin(angle)` = waveform value

```cpp
    gw::core::BufferView view(buffer, 0);
    auto first_quarter = view.subview(0, 128);
    std::cout << "  First sample: " << first_quarter[0] << std::endl;
```

**BufferView Demo:**
- Create view of channel 0
- Create subview of first 128 samples
- Access with subscript operator

```cpp
    gw::core::RingBuffer ring(1024);
    
    float write_data[256];
    for (size_t i = 0; i < 256; ++i) {
        write_data[i] = std::sin(2.0f * 3.14159f * static_cast<float>(i) / 256.0f);
    }
    
    size_t written = ring.write(write_data, 256);
    std::cout << "  Wrote " << written << " samples" << std::endl;
    std::cout << "  Available to read: " << ring.get_available_read() << std::endl;
    
    float read_data[256];
    size_t read_count = ring.read(read_data, 256);
    std::cout << "  Read " << read_count << " samples" << std::endl;
```

**RingBuffer Demo:**
- C-style arrays: `float write_data[256]`
- Generate sine wave
- Write to ring
- Read back
- Show statistics

```cpp
    std::cout << "Milestone 1 complete! Real-time audio buffers are ready." << std::endl;
    return 0;
}
```

---

## Summary & Key Concepts

### Core Components:

1. **AudioFormat** - Stream metadata
   - Sample rate (Hz)
   - Channel count
   - Bit depth

2. **AudioBuffer** - Multichannel storage
   - Planar layout (channels separate)
   - Aligned memory (SIMD optimization)
   - RAII memory management

3. **BufferView** - Non-owning access
   - Lightweight (pointer + size)
   - Subview support
   - Array-like interface

4. **RingBuffer** - Thread-safe FIFO
   - Lock-free (atomics)
   - SPSC only
   - Wraparound circular buffer

### Important C++ Concepts:

**Memory Management:**
- RAII (Resource Acquisition Is Initialization)
- Rule of Five (destructor, copy/move constructors, copy/move assignment)
- `new` / `delete` vs `aligned_alloc` / `free`

**Move Semantics:**
- Transfer ownership without copying
- `&&` = rvalue reference
- `std::move()` = cast to rvalue
- Moved-from objects left in valid but unspecified state

**Const Correctness:**
- `const` parameters (can't modify argument)
- `const` methods (can't modify object)
- `const` pointers (`const float *` vs `float * const`)

**Operator Overloading:**
- `operator==`, `operator!=` (comparison)
- `operator[]` (subscript/indexing)
- `operator=` (assignment)

**Templates and Generics:**
- (Not used in this project, but important for C++)

### Real-Time Audio Principles:

**Predictability:**
- Code must run in bounded time
- No variable-length algorithms
- Pre-allocate all memory

**Thread Safety:**
- Lock-free when possible
- Atomic operations
- Memory ordering

**Performance:**
- Memory alignment for SIMD
- Minimize cache misses
- Avoid allocations in hot path

### Atomic Memory Ordering:

1. **relaxed** - No synchronization (fastest)
   - Use when only one thread accesses

2. **acquire** - See writes before release
   - Use when reading shared data

3. **release** - Make writes visible
   - Use when writing shared data

4. **seq_cst** - Sequential consistency (slowest but safest)
   - Default if not specified

### Lock-Free Ring Buffer Logic:

**Empty:** `read_pos == write_pos`
**Full:** `(write_pos + 1) % capacity == read_pos`
**Available Read:** Distance from read to write
**Available Write:** Capacity - available read - 1

### Build System (CMake):

- `CMakeLists.txt` - Build configuration
- Targets (library, executable)
- Include directories
- Compiler warnings
- Subdirectories

### Testing Philosophy:

- Test each component independently
- Verify basic operations (construct, access, modify)
- Test edge cases (empty, full, wraparound)
- Assert on every condition
- Print progress for debugging

---

This library provides a professional foundation for real-time audio processing, following industry best practices for performance, safety, and maintainability!
