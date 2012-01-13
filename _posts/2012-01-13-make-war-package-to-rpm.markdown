---
layout: post
title: 将java项目打包成rpm格式
excerpt: java项目基本都部署在RHEL平台，rpm格式的文件在RHEL平台上部署非常方便
comments: true
---

以前java web应用是编译生成war包的，war包的管理和升级感觉都不是很方便，最近学习了下linux下面rpm包的管理，感觉挺好的。

现在的流程讲编译好的java文件制作成rpm包，更新的时候只需要使用rpm -ivh **.rpm就行了，卸载更方便rpm -e **.rpm

下面是一个简单的rpm spec文件

{% highlight bash %}
summary: sso login
name: sso
version: 1.0.0
release: 1
source: sso-1.0.0.tar.gz
license: bsd
group: Development/Languages
buildroot: %{_tmppath}/%{name}-%{version}-%{release}
%description
sso is just a demo

%prep

%setup

%build
echo "hello"

%install
rm -rf %{buildroot}
mkdir -p %{buildroot}/var/www/sso.kedou.com
cp -r %{_builddir}/%{name}-%{version}/ %{buildroot}/var/www/sso.kedou.com/

%files

%attr(0755,www,www) /var/www/sso.kedou.com/

%post

%preun

%changelog
* Wed Aug 31 2011 kenny
+ sso-1.0.0-1
- initial package summary: sso is a example
{% endhighlight %}