```
bool isutf8(const void* pbuffer, long size) {
    bool isutf8 = true;
    unsigned char* start = (unsigned char*)pbuffer;
    unsigned char* end = (unsigned char*)pbuffer + size;
    while(start < end) {
        if (*start < 0x80) {
            start++;
        } else if (*start < (0xc0)) {
            isutf8 = false;
            break;
        } else if (*start < (0xe0)) {
            if (start >= end - 1) 
                break;
            if ((start[1] & (0xc0)) != 0x80) {
                isutf8 = false;
                break;
            }
            start += 2;
        } else if (*start < (0xf0)) {
            if (start >= end - 2) 
                break;
            if ((start[1] & (0xc0)) != 0x80 || (start[2] & (0xc0)) != 0x80) {
                isutf8 = false;
                break;
            }
            start += 3;
        } else {
            isutf8 = false;
            break;
        }
    }
    return isutf8;
}
```



