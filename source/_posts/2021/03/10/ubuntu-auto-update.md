---
title: Ubuntuを自動アップデートする
date: 2021-03-10 00:40:47
updated: 2021-09-21 00:31:00
categories: [Ubuntu]
tags:
  - ubuntu
description: "ubuntuを自動アップデートする"
---

### はじめに

Ubuntu 使う際に毎回`sudo apt update && sudo apt upgrade`を打つのがめんどくさいため、自動でアップデートしてくれるようにする。

### 目次

<!-- more -->
<!-- toc -->

### 1./etc/apt/apt.conf.d/20auto-upgrades を編集する

/etc/apt/apt.conf.d/にはパッケージ管理システム`apt`の設定が含まれている。
`20auto-upgrades`を確認しなければならないので一応確認する。
{% codeblock /etc/apt/apt.conf.d/20auto-upgrades lang:bash %}
APT::Periodic::Update-Package-Lists "1"; #自動でパッケージリストをアップデートするか
APT::Periodic::Unattended-Upgrade "1"; #自動でアップデートするか
{% endcodeblock %}

### 2./etc/apt/apt.conf.d/50unattended-upgrades を編集する

自動アップデートの設定だ。自分に合わせ、適所変更してほしい。
{% codeblock /etc/apt/apt.conf.d/50unattended-upgrades lang:diff %}
// Automatically upgrade packages from these (origin:archive) pairs
//
// Note that in Ubuntu security updates may pull in new dependencies
// from non-security sources (e.g. chromium). By allowing the release
// pocket these get automatically pulled in.
Unattended-Upgrade::Allowed-Origins {
"${distro_id}:${distro_codename}";
"${distro_id}:${distro_codename}-security";
// Extended Security Maintenance; doesn't necessarily exist for
// every release and this system may not have it installed, but if
// available, the policy for updates is such that unattended-upgrades
// should also install from here by default.
"${distro_id}ESMApps:${distro_codename}-apps-security";
"${distro_id}ESM:${distro_codename}-infra-security";

- // "${distro_id}:${distro_codename}-updates";
+       "${distro_id}:${distro_codename}-updates";
  // "${distro_id}:${distro_codename}-proposed";
  // "${distro_id}:${distro_codename}-backports";
  };
  // updates までは許可することにした。//を消すことにより、有効化することができる。

// Python regular expressions, matching packages to exclude from upgrading
Unattended-Upgrade::Package-Blacklist {
// The following matches all packages starting with linux-
// "linux-";

    // Use $ to explicitely define the end of a package name. Without
    // the $, "libc6" would match all of them.

// "libc6$";
//  "libc6-dev$";
// "libc6-i686$";

    // Special characters need escaping

// "libstdc\+\+6$";

    // The following matches packages like xen-system-amd64, xen-utils-4.1,
    // xenstore-utils and libxenstore3.0

// "(lib)?xen(store)?";

    // For more information about Python regular expressions, see
    // https://docs.python.org/3/howto/regex.html

};

// This option controls whether the development release of Ubuntu will be
// upgraded automatically. Valid values are "true", "false", and "auto".
Unattended-Upgrade::DevRelease "auto";

// Split the upgrade into the smallest possible chunks so that
// they can be interrupted with SIGTERM. This makes the upgrade
// a bit slower but it has the benefit that shutdown while a upgrade
// is running is possible (with a small delay)
//Unattended-Upgrade::MinimalSteps "true";

// Install all updates when the machine is shutting down
// instead of doing it in the background while the machine is running.
// This will (obviously) make shutdown slower.
// Unattended-upgrades increases logind's InhibitDelayMaxSec to 30s.
// This allows more time for unattended-upgrades to shut down gracefully
// or even install a few packages in InstallOnShutdown mode, but is still a
// big step back from the 30 minutes allowed for InstallOnShutdown previously.
// Users enabling InstallOnShutdown mode are advised to increase
// InhibitDelayMaxSec even further, possibly to 30 minutes.
//Unattended-Upgrade::InstallOnShutdown "false";

// Send email to this address for problems or packages upgrades
// If empty or unset then no email is sent, make sure that you
// have a working mail setup on your system. A package that provides
// 'mailx' must be installed. E.g. "user@example.com"
//Unattended-Upgrade::Mail "";

// Set this value to one of:
// "always", "only-on-error" or "on-change"
// If this is not set, then any legacy MailOnlyOnError (boolean) value
// is used to chose between "only-on-error" and "on-change"
//Unattended-Upgrade::MailReport "on-change";

// Remove unused automatically installed kernel-related packages
// (kernel images, kernel headers and kernel version locked tools).
//Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";

// Do automatic removal of newly unused dependencies after the upgrade
//Unattended-Upgrade::Remove-New-Unused-Dependencies "true";

// Do automatic removal of unused packages after the upgrade
// (equivalent to apt-get autoremove)
//Unattended-Upgrade::Remove-Unused-Dependencies "false";

// Automatically reboot _WITHOUT CONFIRMATION_ if
// the file /var/run/reboot-required is found after the upgrade

- Unattended-Upgrade::Automatic-Reboot "false";
+ Unattended-Upgrade::Automatic-Reboot "true";
  // 自動で適用してほしいので true

// If automatic reboot is enabled and needed, reboot at the specific
// time instead of immediately
// Default: "now"

- Unattended-Upgrade::Automatic-Reboot-Time "02:00";
+ Unattended-Upgrade::Automatic-Reboot-Time "05:00";
  // 再起動する時間を決める。24 時間表記だ

// Use apt bandwidth limit feature, this example limits the download
// speed to 70kb/sec
//Acquire::http::Dl-Limit "70";

// Enable logging to syslog. Default is False
// Unattended-Upgrade::SyslogEnable "false";

// Specify syslog facility. Default is daemon
// Unattended-Upgrade::SyslogFacility "daemon";

// Download and install upgrades only on AC power
// (i.e. skip or gracefully stop updates on battery)
// Unattended-Upgrade::OnlyOnACPower "true";

// Download and install upgrades only on non-metered connection
// (i.e. skip or gracefully stop updates on a metered connection)
// Unattended-Upgrade::Skip-Updates-On-Metered-Connections "true";

// Verbose logging
// Unattended-Upgrade::Verbose "false";

// Print debugging information both in unattended-upgrades and
// in unattended-upgrade-shutdown
// Unattended-Upgrade::Debug "false";

// Allow package downgrade if Pin-Priority exceeds 1000
// Unattended-Upgrade::Allow-downgrade "false";
{% endcodeblock %}

### 3.保存して終了

動作確認をし終了。
