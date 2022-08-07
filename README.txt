ipptool binary for armv7-linux-android

cups-2.3.3 modified with patches from https://github.com/termux/termux-packages/commit/91f43c03e297c70afc6322eef7e26effb1a117cb and
with an addition of lockf to file.c:1008 since r23 bionic libc seems
to doesn't have it.

```
int
lockf (int fd, int cmd, off_t len)
{
  /* lockf is always relative to the current file position.  */
  struct flock fl = {
    .l_type = F_WRLCK,
    .l_whence = SEEK_CUR,
    .l_len = len
  };
  /* lockf() is a cancellation point but so is fcntl() if F_SETLKW is
     used.  Therefore we don't have to care about cancellation here,
     the fcntl() function will take care of it.  */
  switch (cmd)
    {
    case F_TEST:
      /* Test the lock: return 0 if FD is unlocked or locked by this process;
         return -1, set errno to EACCES, if another process holds the lock.  */
      fl.l_type = F_RDLCK;
      if (fcntl (fd, F_GETLK, &fl) < 0)
        return -1;
      if (fl.l_type == F_UNLCK || fl.l_pid == getpid ())
        return 0;
      errno = (EACCES);
      return -1;
    case F_ULOCK:
      fl.l_type = F_UNLCK;
      return fcntl (fd, F_SETLK, &fl);
    case F_LOCK:
      return fcntl (fd, F_SETLKW, &fl);
    case F_TLOCK:
      return fcntl (fd, F_SETLK, &fl);
    }
  errno = (EINVAL);
  return -1;
}
```
