1.yum install subversion

\2. mkdir -p /var/svn/svnrepos

\3. mkdir -p /var/svn/svnrepos

4.cd /var/svn/svnrepos/conf

5.vi passwd

![img](D:\note\img\10)

6.vi authz

![img](D:\note\img\11)

7.vi svnserve.conf

anon-access = read #匿名用户可读

auth-access = write #授权用户可写

password-db = passwd #使用哪个文件作为账号文件

authz-db = authz #使用哪个文件作为权限文件

realm = /var/svn/svnrepos # 认证空间名，版本库所在目录

8.启动

svnserve -d -r /var/svn/svnrepos