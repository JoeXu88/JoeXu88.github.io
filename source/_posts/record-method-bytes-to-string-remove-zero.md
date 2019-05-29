---
title: 'record method bytes to string [remove zero]'
date: 2019-05-21 15:05:40
tags: [c++, string]
categories: program
---


记录一个字节转字符串的方法，主要是去除数字0的方法。因为数字0 会干扰最后字符串的长度，遇到0 会认为字符串结束。  
<!--more -->

```c
std::vector<uint8_t> encode_bytes(const char* bytes, int len)
{
    std::vector<uint8_t> vec(bytes, bytes+len);
    for(uint32_t i=0; i<len; i++)
    {   
        if(bytes[i] == 0)
        {   
            //record postion of 0, and push back to the end of the bytes
            vec[i] = 0x30;
            vec.emplace_back('+');
            uint32_t tmp = i+1; //make sure not 0
            //let n = x*128 + y; so we record x and y
            vec.emplace_back((tmp >> 7) | 0x80); //got x, and make sure not zero, we do x|0x80, assume x <= 0x7f
            vec.emplace_back((tmp%128) | 0x80);  //got y, and make sure not zero, we do y|0x80, actually y <= 127
        }   
    }   

    return vec;
}

std::vector<uint8_t> decode_bytes(const char *str, int expired_len)
{
    //printf("got str len:%d, expired:%d\n", strlen(str), expired_len);
    std::vector<uint8_t> vec(str, str+expired_len);
    int len = strlen(str);

    for(int i=expired_len; i<len-2; i++)
    {   
        if(str[i] == '+')
        { 
            //let n = x*128 + y; so we resume n from x and y
            uint32_t indx = ((str[i+1]&0x7f) << 7) + (str[i+2] & 0x7f);
            vec[indx-1] = 0;
            i += 2;
        }   
    }   

    return vec;
}


//finally, we can use std::string to make string
char *bytes;
//process bytes
std::vector<uint8_t> vec = encode_bytes(bytes, strlen(bytes));
std::string str(vec.begin(), vec.end());
```