*This activity has been created as part of the 42 curriculum by abobeida.*

# get_next_line

## Description

`get_next_line` is a project from the 42 curriculum. Its goal is to implement a C function that reads from a file descriptor and returns one line at a time.

The project focuses on file descriptors, the `read()` system call, static variables, dynamic memory allocation, and memory management.

### Function Prototype

```c
char *get_next_line(int fd);
```

Each call returns the next line, including the newline character (`\n`) if present.

The function returns `NULL` when there is nothing left to read or when an error occurs.

The returned line is dynamically allocated and must be freed by the caller.

## Project Files

| File | Description |
|---|---|
| `get_next_line.c` | Main function and helper functions |
| `get_next_line_utils.c` | String utility functions |
| `get_next_line.h` | Header file containing prototypes and definitions |

## Compilation

Compile the project using:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c main.c -o gnl
```

Run the executable:

```bash
./gnl
```

`BUFFER_SIZE` determines the maximum number of bytes requested in each call to `read()`.

The implementation supports different positive buffer sizes.

## Usage

```c
#include "get_next_line.h"
#include <fcntl.h>
#include <stdio.h>

int main(void)
{
    int fd;
    char *line;

    fd = open("test.txt", O_RDONLY);
    if (fd < 0)
        return (1);
    line = get_next_line(fd);
    while (line)
    {
        printf("%s", line);
        free(line);
        line = get_next_line(fd);
    }
    close(fd);
    return (0);
}
```

The program opens a file in read-only mode, reads and prints one line at a time, frees each returned line, and closes the file when finished.

## Algorithm

The implementation uses a static pointer:

```c
static char *stash;
```

The `stash` stores data that has already been read but has not yet been returned.

Because it is static, the pointer retains its value between calls to `get_next_line`.

It is automatically initialized to `NULL`.

### 1. Reading Data — `ft_read_stash`

The function allocates a temporary buffer of `BUFFER_SIZE + 1` bytes.

It repeatedly calls `read()` until a newline is found in the stash or the end of the file is reached.

After each successful read:

- The buffer is null-terminated.
- The new data is appended to the stash using `ft_strjoin`.
- The previous stash allocation is freed.

If `read()` returns `-1`, the function handles the error and frees the relevant allocated memory.

If `read()` returns `0`, no more data is available from the file descriptor.

### 2. Extracting a Line — `ft_extract_line`

This function extracts the first complete line from the stash.

It copies characters up to and including the first newline character.

If there is no newline, it extracts the remaining characters.

The extracted line is stored in newly allocated memory.

### 3. Updating the Stash — `ft_update_stash`

After extracting a line, this function removes the returned portion from the stash.

If unread characters remain, they are copied into a new string using `ft_substr`.

The previous stash allocation is freed.

If no characters remain, the stash is freed and becomes `NULL`.

### 4. Main Function — `get_next_line`

The main function coordinates the reading, extraction, and updating operations.

It checks the file descriptor and buffer size, reads the required data, extracts one line, updates the stash, and returns the result.

## Utility Functions

| Function | Purpose |
|---|---|
| `ft_strlen` | Calculates the length of a string |
| `ft_strchr` | Searches for a character in a string |
| `ft_strjoin` | Joins two strings into a newly allocated string |
| `ft_substr` | Creates a substring from a given string |

These functions support string manipulation without relying on external string-library functions.

## Memory Management

Memory management is an important part of this project.

The implementation uses `malloc` to allocate memory for the temporary reading buffer, accumulated data, and returned lines.

Temporary allocations and previous stash allocations are freed when they are no longer needed.

The caller is responsible for freeing every returned line.

Allocation failures and read errors must be handled carefully to avoid memory leaks and invalid memory access.

## Edge Cases

The implementation is designed to handle:

- Empty files.
- Files containing a single line.
- Multiple lines.
- Empty lines.
- Lines longer than `BUFFER_SIZE`.
- Files without a final newline.
- Different positive buffer sizes.
- Invalid file descriptors.
- Read errors.

The final line is returned even if it does not end with `\n`.

## Design Decisions

### Why Use a Static Variable?

A call to `read()` may retrieve data belonging to multiple lines.

The extra data must be preserved for the next call.

Using a static stash allows the function to retain this information between calls without using a global variable.

### Why Use Dynamic Memory?

The length of a line is unknown in advance.

Dynamic memory allocation allows the function to handle lines longer than the reading buffer.

### Why Use `ft_strjoin`?

The function combines previously stored data with newly read characters.

This approach is simple to understand and implement, although repeated concatenation can be less efficient for very long lines.

## Limitations

The mandatory implementation uses one static stash pointer.

It is intended for sequential reading from one file descriptor at a time.

Supporting multiple file descriptors independently requires separate stored data for each descriptor, which is addressed in the bonus part of the project.

## Testing

The project can be compiled and tested using different buffer sizes:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=1 get_next_line.c get_next_line_utils.c main.c -o gnl
```

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c main.c -o gnl
```

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=1000 get_next_line.c get_next_line_utils.c main.c -o gnl
```

Memory behavior can be checked using tools such as Valgrind or AddressSanitizer.

### Norminette

```bash
norminette get_next_line.c get_next_line_utils.c get_next_line.h
```

## Resources

- `man 2 read`
- `man 3 malloc`
- `man 3 free`
- [42 Norminette](https://github.com/42School/norminette)

## AI Usage

ChatGPT was used as a learning and review assistant to explain concepts related to file descriptors, static variables, pointer operations, and dynamic memory management.

AI assistance was also used to discuss function decomposition, review potential errors, and improve project documentation.

The author remains responsible for understanding, testing, and validating the final implementation.
