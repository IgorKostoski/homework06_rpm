# RPM Package & Repository - Homework 06

## Assignment
1. Create custom RPM package (Nginx with ngx_brotli module)
2. Create own repository and host it with Nginx

## Environment

| Component | Details |
|-----------|---------|
| Host OS | macOS (Apple Silicon M3) |
| Virtualization | UTM with Apple Virtualization |
| Guest OS | AlmaLinux 9.7 (aarch64) |

## Part 1: Build Custom Nginx RPM

### Install Required Packages
```bash
dnf install -y wget rpmdevtools rpm-build createrepo yum-utils cmake gcc git nano nginx
```

### Download and Build
```bash
mkdir ~/rpm && cd ~/rpm
yumdownloader --source nginx
rpm -Uvh nginx*.src.rpm
yum-builddep -y nginx
```

### Build ngx_brotli Module
```bash
cd /root
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
cd ngx_brotli/deps/brotli
mkdir out && cd out
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF ...
cmake --build . --config Release -j 2 --target brotlienc
```

### Modify Spec and Build RPM
```bash
cd ~/rpmbuild/SPECS/
sed -i 's/--with-http_ssl_module/--add-module=\/root\/ngx_brotli \\\n    --with-http_ssl_module/' nginx.spec
rpmbuild -ba nginx.spec -D 'debug_package %{nil}'
```

### RPM Packages Built
```
nginx-1.20.1-22.el9.3.alma.2.aarch64.rpm
nginx-core-1.20.1-22.el9.3.alma.2.aarch64.rpm
nginx-mod-devel-1.20.1-22.el9.3.alma.2.aarch64.rpm
nginx-mod-http-image-filter-1.20.1-22.el9.3.alma.2.aarch64.rpm
nginx-mod-http-perl-1.20.1-22.el9.3.alma.2.aarch64.rpm
nginx-mod-http-xslt-filter-1.20.1-22.el9.3.alma.2.aarch64.rpm
nginx-mod-mail-1.20.1-22.el9.3.alma.2.aarch64.rpm
nginx-mod-stream-1.20.1-22.el9.3.alma.2.aarch64.rpm
```

## Part 2: Create Repository

### Setup
```bash
yum localinstall -y *.rpm
systemctl start nginx
mkdir /usr/share/nginx/html/repo
cp ~/rpmbuild/RPMS/aarch64/*.rpm /usr/share/nginx/html/repo/
createrepo /usr/share/nginx/html/repo/
```

### Nginx Status
```
nginx version: nginx/1.20.1
Active: active (running)
```

### Repository Listing
```
curl http://localhost/repo/
Index of /repo/ - 11 packages
```

## Part 3: Add to Yum
```bash
cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF
```

### Verify
```bash
yum repolist enabled | grep otus
```
Output: `otus                          otus-linux`

### Install from Repo
```bash
yum install -y percona-release
rpm -qa | grep percona
```
Output: `percona-release-1.0-32.noarch`

## Summary

| Task | Status |
|------|--------|
| Build Nginx with ngx_brotli | ✅ |
| Create RPM packages | ✅ |
| Set up Nginx repository | ✅ |
| Add repository to yum | ✅ |
| Install package from repo | ✅ |
