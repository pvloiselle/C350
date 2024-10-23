# `myar` Program Documentation

## Overview
The `myar` program is a simple archive manipulation tool for Unix-like systems, mimicking the behavior of the standard Unix archive utility. It can list, extract, add, or delete files within an archive. The program supports several command-line options for different operations and operates on archive files that use the standard Unix "ar" format.

## Program Options
- **`-t`**: Displays the table of contents of the archive.
- **`-v`**: Provides verbose output. Used alongside `-t`, `-x`, and `-d` to display detailed file information (e.g., permissions, owner, size, and modification time).
- **`-x`**: Extracts files from the archive.
- **`-o`**: Used in conjunction with `-x`. Restores original file permissions and timestamps during extraction.
- **`-d`**: Deletes specified files from the archive.
- **`-q`**: Quickly appends files to the end of the archive.
- **`-A <N>`**: Adds files modified within the last `N` days to the archive.

### Command Syntax
```
myar [qxotvd:A:] archive-file [file1 ...]
```

- **archive-file**: The archive to operate on.
- **file1 ...**: Files to be manipulated within the archive.

---

## Key Components


### 1. Includes:
1. **`<ar.h>`**: Likely provides functionality related to the archive file format.
2. **`<stdlib.h>`**: General purpose functions such as memory allocation, process control, and conversions.
3. **`<getopt.h>`**: Used to parse command-line options.
4. **`<stdio.h>`**: Standard I/O functions (e.g., `printf`, `fopen`, `fclose`, etc.).
5. **`<unistd.h>`**: Provides access to the POSIX API, including file descriptor manipulation and other system calls.
6. **`<fcntl.h>`**: File control options, such as opening files.
7. **`<string.h>`**: Functions for handling strings (e.g., `strcpy`, `strcmp`).
8. **`<utime.h>`**: Used to set file access and modification times.
9. **`<time.h>`**: Functions to manipulate and format time.
10. **`<sys/stat.h>`**: Provides information about files (e.g., file type, permissions).
11. **`<sys/types.h>`**: Defines data types used in system calls (e.g., `size_t`, `time_t`).
12. **`<dirent.h>`**: Allows directory reading and manipulation.

---

### 2. Constants:
1. **`BUF_MAX_SIZE 8192`**: Defines the maximum size of the buffer used for reading or writing data.
2. **`TIME_BUF_MAX_SIZE 80`**: Defines the maximum size for a buffer to hold formatted time data.
3. **`SECOND_ONE_DAY 86400`**: Represents the number of seconds in a day (24 hours × 60 minutes × 60 seconds).

---


### 3. Error Messages:
The following error messages provide feedback to the user when certain conditions are met:

1. **`ERR_USAGE`**: Displays a usage message when incorrect arguments are passed to the program.
   - Format: `"USAGE: myar [qxotvd:A:] archive-file [file1 .....]\n"`
   
2. **`ERR_OPEN`**: Printed when the archive or file cannot be opened, indicating that it does not exist.
   - Format: `"myar: %s: No such file or directory\n"`

3. **`ERR_INFORMAT`**: Printed when the archive file format is not recognized.
   - Format: `"myar: %s: File format not recognized\n"`

4. **`ERR_MALFORMED`**: Printed when the archive is malformed or corrupt.
   - Format: `"myar: %s: Malformed archive\n"`

5. **`ERR_UNVALID_OP`**: Displayed when more than one operation option is specified, which is not allowed.
   - Format: `"myar: two different operation options specified\n"`

6. **`ERR_NON_REG`**: Printed when the file being processed is not a regular file (e.g., a directory or device file).
   - Format: `"myar: %s: is not regular file\n"`

7. **`ERR_UNVALID_A`**: Used when an invalid argument is passed with the `-A` option.
   - Format: `"myar: -A %s: argument not valid\n"`

8. **`WARN_NO_ENTRY`**: Printed when an entry (file) is not found in the archive.
   - Format: `"no entry %s in archive\n"`

---


### 4. Structure:
#### `struct meta`
This structure holds metadata for files in the archive. It includes:
- **`char name[16]`**: The name of the file (with space for a null terminator).
- **`int mode`**: The file's mode (permissions).
- **`int size`**: The size of the file in bytes.
- **`time_t mtime`**: The last modification time, represented as a `time_t` value (typically a long integer).


---



### 5. Additional Notes:
- The utility seems to support basic archiving operations similar to the traditional UNIX `ar` command.
- The buffer sizes (`BUF_MAX_SIZE`, `TIME_BUF_MAX_SIZE`) help in controlling memory usage while processing files.
- Error handling ensures that the user is given clear feedback when something goes wrong (e.g., invalid input or non-existent files).


---


### 6. **Utility Functions**

#### 1. `int check_ARMAG(int ar_fd, char *buf, const char *ar_fname)`

##### Purpose:
This function checks whether the file referenced by `ar_fd` (archive file descriptor) starts with the expected magic string `ARMAG`, which indicates that the file is a valid archive.

##### Parameters:
- **`ar_fd`**: File descriptor of the archive file to be checked.
- **`buf`**: A buffer to store the magic string read from the archive file.
- **`ar_fname`**: The name of the archive file, used in error messages.

##### Returns:
- **`0`**: If the file's magic string matches `ARMAG`.
- **`1`**: If the magic string does not match `ARMAG`, or if there is an error reading the file.

##### Description:
1. The function reads the first few bytes of the file into the buffer `buf` using `read()`, comparing them to `ARMAG` (a standard archive magic number).
2. If the read operation fails or the string in `buf` does not match `ARMAG`, an error message is printed, and the function returns `1`.
3. If the check passes, the function returns `0`.


#### 2. `int fill_meta(struct ar_hdr hdr, struct meta *meta)`

##### Purpose:
This function populates a `meta` structure with metadata extracted from the archive header `hdr`.

##### Parameters:
- **`hdr`**: A structure of type `ar_hdr` (the archive header), which contains metadata about a file within the archive.
- **`meta`**: A pointer to a `meta` structure that will be filled with the extracted metadata (file name, permissions, size, and modification time).

##### Returns:
- **`0`**: Always returns `0` (although the function currently lacks a return statement).
  
##### Description:
1. **File name extraction**: The function extracts the file name from `hdr.ar_name` and copies it into `meta->name`, ensuring it is properly null-terminated.
2. **Mode parsing**: It parses the file mode (permissions) from `hdr.ar_mode` by converting the last three characters (assumed to be in octal) to decimal permissions and storing them in `meta->mode`.
3. **Size parsing**: It reads the file size from `hdr.ar_size` and stores it in `meta->size`.
4. **Modification time**: It converts `hdr.ar_date` (the modification date stored as a string) into a `time_t` and stores it in `meta->mtime`.


##### Notes:
- The file mode parsing uses hard-coded indices for specific characters, which is marked as needing improvement.
- The function currently lacks a return statement, but it should return `0` upon successful completion.


#### 3. `int fill_ar_hdr(char *filename, struct ar_hdr *hdr)`

##### Purpose:
This function populates an archive header `hdr` for a file specified by `filename`, based on its filesystem metadata.

##### Parameters:
- **`filename`**: The name of the file for which the archive header should be filled.
- **`hdr`**: A pointer to a structure of type `ar_hdr`, where the metadata for the file will be stored.

##### Returns:
- **`0`**: If the header is successfully filled.
- **`1`**: If the file is not found, cannot be opened, or is not a regular file.

##### Description:
1. **File metadata retrieval**: The function retrieves file metadata using the `stat()` system call, storing information such as file size, permissions, and modification time.
2. **File name formatting**: It writes the file name into `hdr->ar_name`, padding or truncating as necessary to fit the field.
3. **Timestamp formatting**: The file's last modification time is formatted into `hdr->ar_date`.
4. **User and group ID formatting**: The user ID (`st_uid`) and group ID (`st_gid`) of the file owner are stored in `hdr->ar_uid` and `hdr->ar_gid`, respectively.
5. **Permissions formatting**: The file's mode (permissions) is formatted into `hdr->ar_mode`.
6. **File size formatting**: The file's size is written into `hdr->ar_size`.
7. **Setting archive magic**: The archive magic string `ARFMAG` is copied into `hdr->ar_fmag`.


##### Notes:
- The function ensures the proper format and padding of metadata fields to meet the archive file format specifications.
- The file must be a regular file (checked using `S_ISREG`), and if not, an error message is displayed.




#### 4. `void print_verbose_file_info(struct ar_hdr my_ar_hdr, struct meta my_meta)`

##### Purpose:
This function prints detailed, verbose information about a file's metadata in a format similar to the output of the `ls -l` command in UNIX. It includes file permissions, owner/group IDs, size, modification time, and file name.

##### Parameters:
- **`my_ar_hdr`**: A structure of type `ar_hdr`, containing metadata fields such as user ID and group ID.
- **`my_meta`**: A structure of type `meta`, containing file metadata such as name, mode (permissions), size, and modification time.

##### Description:
1. **File permissions**: The function first prints the file permissions based on the `mode` field in `my_meta`. It checks for read (`r`), write (`w`), and execute (`x`) permissions for the user, group, and others.
2. **User/Group IDs**: It then prints the user ID and group ID extracted from `my_ar_hdr.ar_uid` and `my_ar_hdr.ar_gid`.
3. **File size**: The file size from `my_meta.size` is printed, formatted to occupy at least six characters.
4. **Modification time**: The modification time from `my_meta.mtime` is formatted and printed using the `strftime()` function. The time is displayed in the format: `Mon DD HH:MM YYYY`.
5. **File name**: Finally, the file name from `my_meta.name` is printed.


##### Output Format:
```
-rwxr-xr-x 1000/1000  12345 Oct 16 12:34 2024 file.txt
```


#### 5. `int read_write_buffer(int in_fd, int out_fd, char *buf, int size)`

##### Purpose:
This function reads data from a file descriptor (`in_fd`) into a buffer and writes the contents of the buffer to another file descriptor (`out_fd`), with a limit on how much data to process in each iteration.

##### Parameters:
- **`in_fd`**: File descriptor for reading data.
- **`out_fd`**: File descriptor for writing data.
- **`buf`**: A buffer to temporarily hold data between reads and writes.
- **`size`**: The total number of bytes to be transferred.

##### Returns:
- **`0`**: If the read and write operations are successful.
- **`1`**: If there is an error during reading or writing.

##### Description:
1. The function determines how much data to read at a time by setting `read_size`, which is the smaller of `size` or `BUF_MAX_SIZE` (a defined constant).
2. It enters a loop, reading data from `in_fd` into `buf` and writing it to `out_fd`. It continues until all `size` bytes are transferred or an error occurs.
3. If the number of bytes read or written does not match the expected `read_size`, or if an I/O operation fails, the function returns `1` to signal an error.
4. The function continues reducing `size` and adjusting `read_size` in each loop iteration until all data is transferred.



#### 6. `int search_name_in_array(const char *name, char **array, int start, int end)`

##### Purpose:
This function searches for a specific `name` in an array of strings, marking it as found by setting the first character of the matching string to a null character (`\0`). It returns whether the `name` was found or not.

##### Parameters:
- **`name`**: The name (string) to search for in the array.
- **`array`**: The array of strings in which the search is performed.
- **`start`**: The starting index for the search.
- **`end`**: The ending index for the search (exclusive).

##### Returns:
- **`1`**: If `name` is found in the array.
- **`0`**: If `name` is not found in the array.

##### Description:
1. The function loops through the `array` from index `start` to `end - 1`.
2. For each string, it compares `name` to the current element in `array` using `strcmp()`.
3. If a match is found, the function sets the first character of the matching string in `array` to `\0` (marking it as "deleted" or found) and sets `hit_flag` to `1`.
4. After completing the search, the function returns `hit_flag`, which indicates whether a match was found (`1`) or not (`0`).



#### 7. `int check_permission(const char *fname)`

##### Purpose:
This placeholder function is intended to check the file permissions of the file specified by `fname`. It is currently incomplete.

##### Parameters:
- **`fname`**: The name of the file whose permissions are to be checked.

##### Returns:
- **`0`**: The current implementation always returns `0`, as the functionality is not yet implemented.


##### Notes:
This function is marked as a `TODO` and may be expanded in the future to actually check file permissions using the `stat()` system call or other methods.


---


### 7. **Operation Functions**

#### 1. `int operation_t(int ar_fd, const char *ar_fname, int verbose)`

##### Purpose:
The function lists the contents of an archive file by reading and parsing archive headers. Depending on the `verbose` flag, it either prints a simple list of filenames or detailed file metadata.

##### Parameters:
- **`ar_fd`**: File descriptor for the archive file to read.
- **`ar_fname`**: The name of the archive file, used for error messages.
- **`verbose`**: If non-zero, prints verbose file information; otherwise, prints only the filenames.

##### Returns:
- **`0`**: On success.
- **`1`**: If there is a read error or a malformed archive header.

##### Description:
1. The function reads the archive headers and verifies the magic string (`ar_fmag`) for each file header.
2. It uses the `fill_meta` function to populate file metadata (`my_meta`) for each file in the archive.
3. Depending on the `verbose` flag, it either prints the detailed file information using `print_verbose_file_info()` or only the filename.
4. If any archive entry has an incorrect format, an error message is printed and the function returns `1`.



---

#### 2. `int operation_x(int ar_fd, const char *ar_fname, char **files_to_extract, int start, int end, char *buf, int x_restore)`

##### Purpose:
The function extracts specific files from an archive, writing them to the filesystem. It also restores file metadata (e.g., permissions and timestamps) if the `x_restore` flag is set.

##### Parameters:
- **`ar_fd`**: File descriptor for the archive file.
- **`ar_fname`**: The name of the archive file.
- **`files_to_extract`**: An array of file names to extract.
- **`start`**: Starting index for the files to extract.
- **`end`**: Ending index for the files to extract.
- **`buf`**: Buffer for reading file data.
- **`x_restore`**: If non-zero, restores original permissions and timestamps.

##### Returns:
- **`0`**: On success.
- **`1`**: On error (e.g., if a file cannot be extracted or written).

##### Description:
1. The function loops through the archive file, reading headers and checking the magic string.
2. For each file, it checks if the name matches any in `files_to_extract`.
3. If a match is found, the file is extracted and written to disk with the option to restore original permissions and timestamps.
4. After processing, the function prints warnings for any files that were not found in the archive.



#### 3. `int operation_x_all(int ar_fd, const char *ar_fname, char *buf, int x_restore)`

##### Purpose:
Extracts all files from the archive, restoring metadata if the `x_restore` flag is set.

##### Parameters:
- **`ar_fd`**: File descriptor for the archive file.
- **`ar_fname`**: The name of the archive file.
- **`buf`**: Buffer for reading file data.
- **`x_restore`**: If non-zero, restores original permissions and timestamps.

##### Returns:
- **`0`**: On success.
- **`1`**: On error.

##### Description:
1. Reads all entries in the archive, checks the header, and extracts each file.
2. Restores metadata for each extracted file if `x_restore` is enabled.
3. The function skips padding between file entries.



#### 4. `int operation_d(int ar_fd, const char *ar_fname, char **files_to_extract, int start, int end, char *buf, int x_restore)`

##### Purpose:
Deletes specified files from an archive and creates a new archive file with the remaining files.

##### Parameters:
- **`ar_fd`**: File descriptor for the original archive file.
- **`ar_fname`**: Name of the archive file.
- **`files_to_extract`**: List of file names to delete from the archive.
- **`start`**: Starting index in the list of files to delete.
- **`end`**: Ending index in the list of files to delete.
- **`buf`**: Buffer for reading and writing file data.
- **`x_restore`**: If non-zero, restores metadata for remaining files.

##### Returns:
- **`0`**: On success.
- **`1`**: On error.

##### Description:
1. Reads each file from the archive and checks if it should be deleted.
2. Writes the remaining files into a new archive.
3. Skips padding as needed.




#### 5. `int operation_q(int ar_fd, const char *ar_fname, char **files_to_extract, int start, int end, char *buf)`

##### Purpose:
Appends files to an existing archive file.

##### Parameters:
- **`ar_fd`**: File descriptor for the archive file.
- **`ar_fname`**: Name of the archive file.
- **`files_to_extract`**: List of files to append to the archive.
- **`start`**: Starting index for the list of files to append.
- **`end`**: Ending index for the list of files to append.
- **`buf`**: Buffer for reading and writing file data.

##### Returns:
- **`0`**: On success.
- **`1`**: On error.

##### Description:
1. Reads the archive file to check its integrity.
2. Appends the files from `files_to_extract` to the archive.
3. Writes the new archive headers and file contents to the archive.



#### 6. `int operation_A(int ar_fd, const char *ar_fname, long N_day, char *buf)`

##### Purpose:
Appends all regular files in the current directory to an archive, except those already in the archive and those modified within the last `N_day` days.

##### Parameters:
- **`ar_fd`**: File descriptor for the archive.
- **`ar_fname`**: Name of the archive file.
- **`N_day`**: Only files modified earlier than `N_day` days are appended.
- **`buf`**: Buffer for reading and writing file data.

##### Returns:
- **`0`**: On success.
- **`1`**: On error.

##### Description:
1. Reads through the archive file to verify its integrity.
2. Opens the current directory and checks for regular files.
3. Appends any file that hasn’t been modified within `N_day` days, and is not already in the archive.


---


### 8. **Main Function**


### `int main(int argc, char *argv[])`

#### Purpose:
This function implements a command-line tool for manipulating archive files. It supports various operations such as extracting, listing, deleting, appending files, and restoring ownership, all controlled via command-line arguments.

#### Parameters:
- `int argc`: The number of arguments passed to the program from the command line.
- `char *argv[]`: An array of strings representing the command-line arguments.

#### Return Value:
- Returns `0` on success or `1` if an error occurs (e.g., invalid arguments, file access issues).

#### External Variables:
- `char *optarg`: Stores the argument value for command-line options that require an argument (used with `getopt`).
- `int optind`: Tracks the index of the next element in `argv[]` to be processed by `getopt`.

#### Internal Variables:
- `int opt`: Stores the current command-line option being processed.
- `char copt`: Tracks the currently specified operation ('x', 't', 'd', 'q', or 'A').
- `char *ar_fname`: Stores the name of the archive file to be manipulated.
- `int verbose`: Flag indicating whether verbose mode (`-v`) is active.
- `int x_restore`: Flag indicating whether to restore ownership during extraction (`-o`).
- `int ar_fd`: File descriptor for the archive file.
- `int ret`: Stores the return status from operations.
- `char *buf`: Buffer for reading and writing file data, allocated at runtime.
- `long N_day`: Stores the number of days for the `-A` operation, where files older than `N_day` are appended to the archive.
- `char *cptr`: Helper pointer for parsing `N_day`.

#### Supported Command-Line Options:
- **`-t`**: List the contents of the archive.
- **`-v`**: Enables verbose output for listing (`-t`) or extraction (`-x`).
- **`-x`**: Extracts files from the archive. If no files are specified, all files are extracted.
- **`-o`**: Restores the original ownership and permissions during extraction.
- **`-d`**: Deletes specified files from the archive.
- **`-q`**: Appends files to the archive.
- **`-A <N>`**: Appends files older than `N` days to the archive.

#### Function Workflow:
1. **Command-line Parsing**: The function uses `getopt` to parse the command-line options. Supported options are `qxotvdA:`.
   - Each option is processed and stored in variables for later use.
   - If an invalid combination of options is provided, the function prints an error message and returns `1`.

2. **File Handling**:
   - After parsing options, the function checks whether an archive file name has been provided. If not, an error is displayed, and the function exits.
   - The archive file is opened with `open()`. If the file cannot be opened, an error message is printed, and the function returns `1`.

3. **ARMAG Check**:
   - The function checks the ARMAG magic string to verify that the file is a valid archive.
   - Based on the specified operation, either extraction, listing, deletion, appending, or restoration logic is executed.

4. **Operation Execution**:
   - **`-x` (Extraction)**: Extracts files from the archive. If no files are specified, all files are extracted.
   - **`-t` (Listing)**: Lists the contents of the archive. If verbose mode (`-v`) is enabled, detailed information is displayed.
   - **`-d` (Deletion)**: Deletes specified files from the archive.
   - **`-q` (Appending)**: Appends new files to the archive.
   - **`-A <N>` (Appending older files)**: Appends files older than `N` days to the archive.

5. **Cleanup**: The allocated buffer (`buf`) is freed before returning the result.

#### Error Handling:
- **Invalid Command-Line Options**: The function checks for invalid combinations of options and prints relevant error messages.
- **Missing Archive File**: If no archive file is specified, the function prints a usage error.
- **File Access Errors**: If the archive file cannot be opened or written to, an error message is displayed.


---

### 9. **Limitations and Future Work**
- Hardcoded values for permission bits need improvement (marked as "TODO").
- Permission checking and error handling could be more robust (also marked as "TODO").
- Additional error messages may be required for better debugging.

---

### 10. **Example Usage**

1. **List files in the archive**:
   ```
   myar -t archive.a
   ```

2. **Extract a file**:
   ```
   myar -x archive.a file1.txt
   ```

3. **Delete files from the archive**:
   ```
   myar -d archive.a file1.txt file2.txt
   ```

4. **Append files to the archive**:
   ```
   myar -q archive.a file3.txt
   ```

5. **Add files modified within the last 7 days**:
   ```
   myar -A 7 archive.a
   ```
