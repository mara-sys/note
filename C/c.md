# 1. C 遍历路径及文件
&emsp;&emsp;在本文中，我们探讨如何递归遍历 C 中的文件夹。我们使用 dirent.h 库以及 readdir, opendir 和 closedir 等基本函数来实现这一点。

## 1.1 Streams 和 Files
&emsp;&emsp;stream 可以分为两种类型：
1. text streams
2. binary streams

* Text streams: 在文本流中，发生了字符转换。 这意味着如果我们在文件中的 \n 不显示为 \n 而是显示为换行。 这意味着在读取过程中，实际值会有所变化，并且会显示修改后的数据。
* Binary streams: 但是在二进制流中，不会发生字符转换。 这意味着如果我们将文件中的 \n 显示为 \n 而不是将其显示为换行。 这意味着在读取过程中，显示的所有数据都不会修改实际值。

&emsp;&emsp;c 没有提供任何直接的文件处理和遍历方法，因此我们需要使用头文件：
```c
#include <dirent.h>
```
1. 这个头文件中使用 typedef 定义了类型`DIR`。此 DIR 表示流的目录。
2. 它还定义了一个结构体，结构体的成员是一个字符数组（用于存储文件名）和一个用于存储文件序列号的附加字段（ino_t）。

&emsp;&emsp;文件遍历中常用的函数：
1. readdir()
2. opendir()
3. closedir()

### 1.1.1 readdir()
&emsp;&emsp;函数原型：
```c
struct dirent *readdir(DIR *dirp);
```
&emsp;&emsp;什么是 DIR 数据类型？
&emsp;&emsp;如前所述，C 借助流进行交互，而 DIR 是流交互的一种方式。 此数据类型应始终与指针一起使用，因为与 DIR 的直接实例交互可能会在以后导致错误。
&emsp;&emsp;参数的类型为 DIR 指针，表示目录流中的当前位置，当调用此调用时，此函数返回将指向流交互中下一个目录的指针的位置。
&emsp;&emsp;当我们到达文件目录的末尾时，它返回空指针。
&emsp;&emsp;示例：
```c
#include <stdio.h>
#include <dirent.h>
#include <string.h>

int main()
{
    DIR *dir;
    struct dirent *dp;
    char * file_name;
    dir = opendir("/Users/babayaga/Desktop/foldertest");
    while ((dp=readdir(dir)) != NULL) {
        
        file_name = dp->d_name; 
        printf("The File Name :-  \"%s\"\n",file_name);
    }
    closedir(dir);
    return 0;
}
```

### 1.1.2 opendir()
&emsp;&emsp;函数原型：
```c
DIR *opendir(const char *fold_name);
```
&emsp;&emsp;传递的参数是目录名称，并将其设为 const，以便函数不会因为某些错误而修改目录。
&emsp;&emsp;成功时，它返回一个 DIR 类型的指针，否则它返回 null。

### 1.1.3 closedir()
&emsp;&emsp;函数原型：
```c
int closedir(DIR *dirp);
```
&emsp;&emsp;此功能用于关闭已打开的目录。 如果关闭成功则返回 0 否则发生错误则返回 -1。

### 1.1.4 结构体类型
```c
struct dirent
{
    long d_ino;                 /* inode number  索引节点号 */
    off_t d_off;                /* offset to this dirent 在目录文件中的偏移 */
    unsigned short d_reclen;    /* length of this d_name 文件名长 */
    unsigned char d_type;       /* the type of d_name 文件类型 */
    char d_name [NAME_MAX+1];   /* file name (null-terminated) 文件名，最长256字符 */
}
```
```c
struct __dirstream
{
    void *__fd;                     /* `struct hurd_fd' pointer for descriptor.   */
    char *__data;                   /* Directory block.   */
    int __entry_data;               /* Entry number `__data' corresponds to.   */
    char *__ptr;                    /* Current pointer into the block.   */
    int __entry_ptr;                /* Entry number `__ptr' corresponds to.   */
    size_t __allocation;            /* Space allocated for the block.   */
    size_t __size;                  /* Total valid data in the block.   */
    __libc_lock_define (, __lock)   /* Mutex lock for this structure.   */
};
typedef struct __dirstream DIR;
```

### 1.1.5 示例
#### 1.1.5.1 示例代码 1
&emsp;&emsp;在这段代码中，我们将简单地打印我们作为参数传递的目录路径的内容。 此代码不会进一步探索子目录。
```c
#include <stdio.h> 
#include <dirent.h> 
  
int main(void) 
{ 
    struct dirent *de;  // Pointer for directory entry 
  
    // opendir() returns a pointer of DIR type.  
    char ch[1000000];
    scanf("%s",ch);// this is added to take input the path of directory to be searched
    DIR *dr = opendir(ch); 
  
    if (dr == NULL)  // opendir returns NULL if couldn't open directory 
    { 
        printf("Could not open current directory" ); 
        return 0; 
    } 

    while ((de = readdir(dr)) != NULL) 
            printf("%s\n", de->d_name); 
  
    closedir(dr);     
    return 0; 
} 
```

##### 1.1.5.2 示例代码 2
&emsp;&emsp;在这段代码中，我们将递归地打印每个目录的内容，如果遇到目录，也会进一步探索该目录。
&emsp;&emsp;简而言之，它将列出从给定位置开始的所有文件。
```c
#include <stdio.h>
#include <string.h>
#include <dirent.h> 

void myfilerecursive(char *path);


int main()
{
   
    char name[100000]; 
    printf("Enter path to list files: ");
    scanf("%s", name);

    myfilerecursive(name);

    return 0;
}

void myfilerecursive(char *basePath)
{
    char path[1000];
    struct dirent *dp;
    DIR *dir = opendir(basePath);

   
    if (!dir)
        return;

    while ((dp = readdir(dir)) != NULL)
    {
        if (strcmp(dp->d_name, ".") != 0 && strcmp(dp->d_name, "..") != 0)
        {
            printf("%s\n", dp->d_name);
            strcpy(path, basePath);
            strcat(path, "/");
            strcat(path, dp->d_name);

            myfilerecursive(path);
        }
    }

    closedir(dir);
}
```














