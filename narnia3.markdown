This is a solution guide to the Narnia3 Level at 
http://overthewire.org/wargames/narnia/

This write-up was created on 28 February 2015.

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
 
// ... code removed for brevity
 
        exit(1);
}
```

This is an interesting problem because the program accepts multiple command line arguments.
This includes a strcpy of the 2nd argument in the command line to a buffer called ifile.
We can probably do something interesting with that.
