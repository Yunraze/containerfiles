# PowerShell Container

Pretty simple: Runs `pwsh` in a Debian container.

Compile and run the container.

**Default user:** `user`

**NOTE**: For SELinux distributions, use the `:z` appendix on the volume mount 
to make sure SELinux doesn't block you from using the mount.

```bash
podman run --rm -it --user user -v $(pwd):/workspace:z 
--workdir /workspace [CONTAINER NAME]:[TAG]
```
