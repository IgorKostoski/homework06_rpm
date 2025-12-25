# RPM Package & Repository - Homework 06

## Assignment
1. Create custom RPM package (Nginx with ngx_brotli module)
2. Create own repository and host it with Nginx

## Environment
- Host OS: macOS (Apple Silicon M3)
- Virtualization: UTM
- Guest OS: AlmaLinux 9.7 (aarch64)

## Tasks Completed
- Built Nginx with ngx_brotli module
- Created RPM packages
- Set up Nginx repository
- Added repository to yum
- Installed package from custom repo

## Key Commands
- yumdownloader --source nginx
- rpmbuild -ba nginx.spec
- createrepo /usr/share/nginx/html/repo/
- yum install percona-release
