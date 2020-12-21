+++
title = "Benchmarking C++ Bitstream Implementations"
date = "2020-12-21T00:46:21Z"
description = "Benchmarking c++ bitstream implementations"
tags = ["bitstream", "c++", "benchmarking"]
+++

---

I've recently been working on some projects that required writing 
arbitrary numbers of bits to a byte array. Processors work with bytes not bits,
so some bit manipulation is needed to convert writing bits
to writing bytes. Examples of where this might come up could be in writing 
out a Huffman code or in serializing compressed data.

Relatively simple implementations of writing arbitrary number of bits exist,
but I wanted to experiment and see what is the fastest way to write arbitrary
streams of bits on modern processors.

First a reference implementation that respects byte boundaries, not designed to be
particularly fast:

```c++
#include <cstdint>

class BitWriter
{
public:
  BitWriter(std::uint8_t* buffer, std::uint64_t bufferLen)
    : m_buffer(buffer)
    , m_numBits(0) {}

  bool write(std::uint8_t bits, std::uint8_t numBits)
  {
    if (numBits == 0)
    {
      return true;
    }

    // either we have space to write into our buffer or not
    std::uint8_t bitsWritten = (m_numBits % 8);
    std::uint8_t availBits = 8 - bitsWritten;
    if (numBits > availBits)
    {
      // write availBits
      unsafe_write(bits, availBits);
      return (write(bits >> availBits, numBits - availBits));
    }
    else
    {
      unsafe_write(bits, numBits);
      return true;
    }
  }

private:
  inline bool unsafe_write(std::uint8_t bits, std::uint8_t numBits)
  {
    uint8_t* byte = m_buffer + (m_numBits / 8);
    std::uint8_t mask = static_cast<std::uint8_t>((1 << numBits) - 1);
    *byte |= (bits & mask) << (m_numBits % 8);
    m_numBits += numBits;
  }

  std::uint8_t* m_buffer;
  std::uint64_t m_numBits;
};
```
