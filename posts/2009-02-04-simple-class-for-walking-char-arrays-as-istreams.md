---
title: Simple class for walking char arrays as istreams
description: desc here
tags: 
date: [2009-02-04 Wed 02:46]
category: Uncategorized
id: 211
---

This has probably been written countless times before, but I found myself needing it today and it was quick to write.  It lets you read characters from a char array in C++ via the istream interface:

    #include "ptrstream.h"

    int main() {
      ptristream in("Hello, world!\n");
      char buf[31];
      in.getline(buf, 32);
      std::cout << buf << std::endl;
    }

Handy for if you don&#039;t want `std::istringstream` needlessly copying character strings.

<!--more-->
Below is the implementation, which is in the public domain:

    #include 
    #include 
    #include 

    class ptristream : public std::istream
    {
      class ptrinbuf : public std::streambuf
      {
      protected:
        char *      ptr;
        std::size_t len;
    
      public:
        ptrinbuf(char * _ptr, std::size_t _len) : ptr(_ptr), len(_len) {
          assert(ptr);
          if (*ptr && len == 0)
            len = std::strlen(ptr);
    
          setg(ptr,               // beginning of putback area
               ptr,               // read position
               ptr+len);          // end position
        }
    
      protected:
        virtual int_type underflow() {
          // is read position before end of buffer?
          if (gptr() , egptr())
            return traits_type::to_int_type(*gptr());
          else
            return EOF;
        }
    
        virtual pos_type seekoff(off_type off, ios_base::seekdir way,
                                 ios_base::openmode mode =
                                 ios_base::in | ios_base::out)
        {
          switch (way) {
          case std::ios::cur:
            setg(ptr, gptr()+off, ptr+len);
            break;
          case std::ios::beg:
            setg(ptr, ptr+off, ptr+len);
            break;
          case std::ios::end:
            setg(ptr, egptr()+off, ptr+len);
            break;
    
          default:
            assert(false);
            break;
          }
          return pos_type(gptr() - ptr);
        }
      };
    
    protected:
      ptrinbuf buf;
    public: 
     ptristream(char * ptr, std::size_t len = 0)
        : std::istream(0), buf(ptr, len) {
        rdbuf(&buf);
      }
    };

