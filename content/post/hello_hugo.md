+++
date = "2017-01-06T16:58:20Z"
title = "在CentOS上用hugo搭建静态blog"
draft = false
+++

# 使用源码部署hugo
1. 部署golang
<pre><code>
yum install golang
</code></pre>

2. 下载hugo源代码
<pre><code>
  wget https://github.com/spf13/hugo/archive/v0.18.1.tar.gz
  tar xf v0.18.1.tar.gz
  cd hugo-0.18.1/
  export GOPATH=/usr/local/go
  go get -v github.com/spf13/hugo
</code></pre>

# hugo的使用
1. 创建项目
<pre><code>
  /usr/local/go/bin/hugo new site myblogs
</code></pre>
2. 创建第一个页面
<pre><code>
  /usr/local/go/bin/hugo new post/first.md
</code></pre>
3. 使用themes
<pre><code>
  cd themes/
  git clone https://github.com/kakawait/hugo-tranquilpeak-theme.git
  cp hugo-tranquilpeak-theme/exampleSite/* ..
</code></pre>

# 启动服务
1. 使用supervisor来启动服务，需要配置配置文件'/etc/supervisor/conf.d/hugo.conf'
<pre><code>
  [program:hugo]
  user=blogs
  directory=/opt/myblogs
  command=/usr/local/go/bin/hugo server --watch --baseURL="http://[Your_blog_DNS]" --appendPort=false --disableLiveReload
  process_name=%(program_name)s
  autostart=true
  redirect_stderr=true
  stdout_logfile=/var/log/hugo/myblogs.log
  stdout_logfile_maxbytes=10MB
  stdout_logfile_backups=0
</code></pre>

2. 启动supervisord服务
<pre><code>
  service supervisord start
</code></pre>

服务的启停可以通过supervisorctl来完成
<pre><code>
  supervisorctl stop hugo
  supervisorctl start hugo
</code></pre>

启动后hugo通常会在本地127.0.0.1:1313上面运行

3. 配置nginx
<pre><code>
  $ cat /etc/nginx/conf.d/blogs.conf
  server {
    listen       80;
    listen       [::]:80;
    server_name  [Your_blog_DNS];
    root /opt/myblogs;
    location ^~ / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   Host      $http_host;
        proxy_pass         http://127.0.0.1:1313/;
    }
  }
</code></pre>

至此，系统的搭建基本完成，赶紧通过http://[Your_blog_DNS]来访问吧 :-)


But, 你很快会发现本例中使用theme包含若干的菜单访问后出现404页面，原来该theme的exampleSite没有提供默认的菜单内容，造成'content'目录中不存在对应的内容...

好吧，让我们来完成最后一步

4. 构造你自己的URL
<pre><code>
  cd content
  mkdir archives
  touch archives/_index.md
  mkdir categories
  touch categories/_index.md
  mkdir tags
  touch tags/_index.md
</code></pre>

此时链接都已经可以访问啦！
<pre><code>
  http://[Your_blog_DNS]/archives
  http://[Your_blog_DNS]/categories
  http://[Your_blog_DNS]/tags
</code></pre>
