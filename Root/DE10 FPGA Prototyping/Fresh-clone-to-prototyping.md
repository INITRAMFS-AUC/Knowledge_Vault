Below are the steps to run the simulation after cloning the KWS-SoC repo.

**Set up `PATH` to point to `Quartus` binaries**. You can add it temporarilty:
```bash
export PATH=$PATH:/home/public/Quartus-Installed/quartus/bin
```
Or add it permanently by opening the `~/.zshrc` file

Add the following line to the end of the file:
```bash
export PATH=$PATH:/home/public/Quartus-Installed/quartus/bin
```

**Set up `PATH` to point to `riscv-toolchain` binaries**.

Add the following line to the end of the `~/.zshrc` file:
```bash
export PATH=$PATH:/opt/riscv/gcc15/bin
```

**Install prerequisites**

```bash
sudo apt install -y yosys clang gtkwave
```

**Create publick key**
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
and then add the identity
```bash
ssh-add ~/.ssh/id_ed25519
```
then print it to copy and add to github
```bash
cat ~/.ssh/id_ed25519.pub
```

**Update gitsubmodules to clone hazard**

```bash
git submodule update --init --recursive
```