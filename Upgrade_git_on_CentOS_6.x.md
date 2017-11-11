### 1、安装rpmforge软件库（http://repoforge.org/use/）
  使用root权限执行：
  <pre><code># rpm -i 'http://repository.it4i.cz/mirrors/repoforge/redhat/el6/en/x86_64/rpmforge/RPMS/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm'
# rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt</code></pre>

### 2、启用rpmforge软件库
  打开文件/etc/yum.repos.d/rpmforge.repo，找到[rpmforge-extras]，把enabled=0改成enabled=1
### 3、升级git
   <pre><code>yum update git</code></pre>
   
   直接安装（不用编辑repo配置文件）：<pre><code>yum --disablerepo=base,updates --enablerepo=rpmforge-extras install git</code></pre>
   直接更新（不用编辑repo配置文件）：<pre><code>yum --disablerepo=base,updates --enablerepo=rpmforge-extras update git</code></pre>
### 4、关闭rpmforge软件库
   升级完成后，关闭rpmforge-extras库。与步骤2.2类似，用文本编辑器打开/etc/yum.repos.d/rpmforge.repo，找到[rpmforge-extras]，把enabled=1改成enabled=0   
   清理缓存：<code>yum clean all</code>
