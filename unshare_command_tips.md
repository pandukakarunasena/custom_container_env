
To ensure proper isolation, you may want to use the `unshare` command with the `-r` or `--mount-proc` option, which will create a new mount namespace with a private `/proc` mount. This can help in isolating the process view:

```bash
unshare --uts --pid --net --mount --ipc --fork --mount-proc /bin/bash
```

Adding `--mount-proc` instructs `unshare` to mount a private `/proc` inside the new namespace, effectively isolating the process information.

After running this command, if you execute `ps aux` within the isolated environment, you should only see processes within that environment, not those from the host machine.



The `--mount` and `--mount-proc` options in the `unshare` command relate to creating a new mount namespace, but they serve different purposes:

1. **--mount:**
   - When you use `--mount` with `unshare`, it creates a new mount namespace for the isolated environment.
   - This means that filesystem mounts within this namespace will be independent of the global system.

2. **--mount-proc:**
   - `--mount-proc` is a specific option that can be used with `--mount` in the context of `unshare`.
   - When you include `--mount-proc`, it additionally mounts a private `/proc` filesystem inside the new namespace. This isolates the process information, making the processes within the namespace independent of the global `/proc`.

Here's an example combining both options:

```bash
unshare --mount --mount-proc /bin/bash
```

This command creates a new process with its own mount namespace and mounts a private `/proc` filesystem inside that namespace. The `--mount` option is generally used when you want to isolate filesystem mounts, and `--mount-proc` is used when you want to isolate the `/proc` information.

In contrast, using `--mount` alone without `--mount-proc` would create a new mount namespace but wouldn't isolate the `/proc` information, so processes might still be visible across namespaces when inspecting `/proc` in the host environment.