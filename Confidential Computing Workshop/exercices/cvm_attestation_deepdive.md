# Fetching and verifying raw sev-snp report

Welcome to this comprehensive guide to installing tpm2-tss from source and verifying raw AMD SEV-SNP manually on Linux. Throughout this guide, you will follow a step-by-step process to set up tpm2-tss, a TPM2.0 Software Stack, in your environment, leveraging its unique security features. Furthermore, you will also learn to fetch and verify raw AMD SEV-SNP manually.

It is worth mentioning that the material presented here is largely inspired by the resources available at  [GitHub-TPM2-TSS](https://github.com/tpm2-software/tpm2-tss/blob/master/INSTALL.md) and [Cvm-guest-attestation · GitHub](https://github.com/Azure/confidential-computing-cvm-guest-attestation/blob/main/cvm-guest-attestation.md) . This lab leverages that content, and you can refer to this resource to delve deeper.

## Installing the tpm2-tss [Github-TPM2-TSS](https://github.com/tpm2-software/tpm2-tss/blob/master/INSTALL.md)
## Dependencies
To build and install the tpm2-tss software, the following software packages are required. As many dependencies are platform-specific, the sections below describe them for the supported platforms.


## GNU/Linux:
* GNU Autoconf
* GNU Autoconf Archive, version >= 2019.01.06
* GNU Automake
* GNU Libtool
* C compiler
* C library development libraries and header files
* pkg-config
* doxygen
* OpenSSL development libraries and header files, version >= 1.1.0
* libcurl development libraries
* Access Control List utility (acl)
* JSON C Development library
* Package libusb-1.0-0-dev


The following are dependencies only required when building test suites.
* Integration test suite (see ./configure option --enable-integration):
    - uthash development libraries and header files
    - ps executable (usually in the procps package)
    - ss executable (usually in the iproute2 package)
    - tpm_server executable (from https://sourceforge.net/projects/ibmswtpm2/)
* Unit test suite (see ./configure option --enable-unit):
    - cmocka unit test framework, version >= 1.0
* Code coverage analysis:
    - lcov

Most users will not need to install these dependencies.

### Clone the repo 

```
git clone https://github.com/tpm2-software/tpm2-tss.git && cd tpm2-tss/
```

### Ubuntu
```
sudo apt -y update && sudo apt -y install \
  autoconf-archive \
  libcmocka0 \
  libcmocka-dev \
  procps \
  iproute2 \
  build-essential \
  git \
  pkg-config \
  gcc \
  libtool \
  automake \
  libssl-dev \
  uthash-dev \
  autoconf \
  doxygen \
  libjson-c-dev \
  libini-config-dev \
  libcurl4-openssl-dev \
  uuid-dev \
  libltdl-dev \
  libusb-1.0-0-dev \
  libftdi-dev
```
Note: In some Ubuntu versions, the lcov and autoconf-archive packages are incompatible with each other. It is recommended to download autoconf-archive directly from upstream and copy `ax_code_coverage.m4` and `ax_prog_doxygen.m4` to the `m4/` subdirectory of your tpm2-tss directory.


# Building From Source
## Bootstrapping the Build
To configure the tpm2-tss source code first run the bootstrap script, which
generates list of source files, and creates the configure script:
```
./bootstrap
```

Any options specified to the bootstrap command are passed to `autoreconf(1)`.

## Configuring the Build

Then run the configure script, which generates the makefiles:
```
./configure
```

## Compiling the Libraries
Then compile the code using make:
```
make -j$(nproc)
```

## Installing the Libraries
Once you've built the tpm2-tss software it can be installed with:
```
sudo make install
```

This process will install the libraries to a location determined at the time of configuration. You can see the available options in the output of `./configure --help`. Typically, you won't need to do much more than providing an alternative `--prefix` option at the time of configuration. You might also need to provide the `DESTDIR` at the time of installation if you're packaging.


# Post-install

## udev
Once you've installed this udev rule in the appropriate location for your distribution, you need to instruct udev to reload its rules and apply the new one. This can typically be accomplished with the following command:


```
sudo udevadm control --reload-rules && sudo udevadm trigger
```

If this doesn't work on your distro please consult your distro's
documentation for UDEVADM(8).

## ldconfig

It may be necessary to run `ldconfig` (as root) to update the run-time bindings before executing a program that links against `libsapi` or a TCTI library:

```
sudo ldconfig
```

# Fetching and verifying raw AMD SEV-SNP manually

# Linux  [Cvm-guest-attestation · GitHub](https://github.com/Azure/confidential-computing-cvm-guest-attestation/blob/main/cvm-guest-attestation.md)

1. For cryptographic verification, install the `tpm2-tss` library, which is an open source TPM software stack widely used across many OS distributions supported by CVMs. You can find the installation instructions for relevant OS distributions at the following link:
    - [tpm2-tss installation guide](https://github.com/tpm2-software/tpm2-tss/blob/master/INSTALL.md)
        - For Ubuntu OS, please use the updated instructions available at the Canonical site - [Ubuntu tpm2-tss](https://packages.ubuntu.com/source/focal-updates/tpm2-tss). Download the `tpm2-tss` zipped archive and follow the instructions in the `INSTALL.md` file.
    - After completing the above steps, install `tpm2-tools`.

```bash
sudo apt install tpm2-tools
```
2. Obtain the VCEK certificate by executing the following command. This command retrieves the certificate from a [well-known IMDS endpoint](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service?tabs=windows):


```bash
curl -H Metadata:true http://169.254.169.254/metadata/THIM/amd/certification > vcek
```

```
sudo snap install jq
```

```
cat ./vcek | jq -r '.vcekCert , .certificateChain' > ./vcek.pem
```

3. Execute the following command to extract the raw SNP report from the vTPM NVRAM. This report was pre-generated by the HCL at boot and will be dumped in binary form. Note that this is **not** a dynamically generated report.


```    
sudo tpm2_nvread -C o 0x01400001 > ./snp_report.bin
```
```
dd skip=32 bs=1 count=1184 if=./snp_report.bin of=./guest_report.bin
```

```
mkdir certs && mv guest_report.bin certs && mv vcek.pem certs
```

**Note**: Once the VCEK certificate and SNP report are available, the following steps can be carried out on any other trusted CVM or non-CVM machine.


## Downloading the SEV-Tool ([GitHub – AMDESE/sev-tool: AMD SEV Tool](https://github.com/AMDESE/sev-tool))
1. Boot into a kernel that supports SEV. Refer to the previous sections to confirm if your kernel supports SEV.
2. Install the dependencies: `git`, `make`, `gcc`, `g++`, and `openssl`.

  
     ```sh
     sudo apt install git make gcc g++ -y --allow-unauthenticated
     ```
     
3. The Github is located at: [SEV-Tool Github](https://github.com/AMDESE/SEV-Tool).
     ```sh
     git clone https://github.com/AMDESE/sev-tool.git
     ```
4. Compile the SEV-Tool.
   - Running the build script accomplishes the following tasks:
      - Downloads, configures, and builds the OpenSSL Git code (utilizing submodule init/update).
      - Cleans and builds the SEV-Tool.
   - To execute the build script: SEV-Tool
   - To run the build script
     ```sh
     cd sev-tool
     autoreconf -vif && ./configure && make && cp src/sevtool .
     ```

## How to Run the SEV-Tool

 1. Use the AMD SEV-Tool to parse the report and verify that SNP is enabled. **Note**: The `sevtool` binary is located in the `<repo root>/src` folder.


```
sudo ./sevtool --ofolder ../certs --validate_guest_report
```
If you see the output described below, it indicates that the **SNP report** is valid, the CVM is operating on a genuine **AMD processor** and that **SNP** is enabled for confidentiality on this specific processor.


```
Guest report validated successfully! 
```
