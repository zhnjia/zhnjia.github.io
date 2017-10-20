---

layout: page
title: Read Mail on Emacs with mu4e

---


# Table of Contents

1.  [Rmail](#org3a72c2c)
2.  [mu4e](#org847464f)
    1.  [下载邮件](#orgd8285f6)
    2.  [查看邮件](#org0e3d134)
    3.  [Key Bindings](#orgb2eb34b)
3.  [enjoying!](#org34c5331)


<a id="org3a72c2c"></a>

# Rmail

emacs自带的Rmail是可以阅读邮件的，配合movemail程序，用很简单的配置就可以实现：

    ;; emacs内置的movemail不支持imap，可以安装gnu mailutils提供的movemail以支持imap
    (setq rmail-movemail-program "/usr/bin/movemail")

    ;; 设置imap服务
    (setq rmail-primary-inbox-list '("imap://me@somemail.com")
          rmail-remote-password-required t
          rmail-remote-password "xxxxxxxx"
          )

使用rmail命令就可以看到邮件了，rmail会调用movemail下载邮件，并展示一封邮件。但是Rmail的展示以及操作不太友好，于是就有了mu4e(mu for emacs).
[Referrence of Rmail](https://www.gnu.org/software/emacs/manual/html_node/emacs/Rmail.html#Rmail)


<a id="org847464f"></a>

# mu4e

"mu4e is an emacs-based e-mail client. It’s based on the mu e-mail indexer/searcher. It attempts to be a super-efficient tool to withstand the daily e-mail tsunami."
-&#x2014; [emacswiki](https://www.emacswiki.org/emacs/mu4e)


<a id="orgd8285f6"></a>

## 下载邮件

我选择使用offlineimap下载邮件，vim时期配置过mutt，当时就用的offlineimap，觉得蛮好用的。
offlineimap的配置网上也是很多，比如我自己的配置(~/.offlineimaprc)：

    [general]
    accounts = mymail
    maxsyncaccounts = 3

    [Account mymail]
    localrepository = mymail-local
    remoterepository = mymail-remote
    status_backend = sqlite

    [Repository mymail-local]
    type = Maildir
    localfolders = ~/.mailbox

    [Repository mymail-remote]
    type = Gmail
    folderfilter = lambda folder: folder in ['[Gmail]/Archive', '[Gmail]/All Mail', '[Gmail]/Drafts', '[Gmail]/Sent Mail', '[Gmail]/Starred']
    nametrans = lambda foldername: re.sub ('^\[gmail\].', '',
                                   re.sub ('all_mail', 'all',
                                   re.sub ('sent_mail', 'sent',
                                   re.sub ('starred', 'flagged',
                                   re.sub (' ', '_', foldername.lower())))))
    remoteuser = jiazhang@gmail.com
    remotepass = <gmail app password>
    ssl = yes
    maxconnections = 1
    realdelete = no
    sslcacertfile = /etc/ssl/certs/ca-certificates.crt

配置好，在终端跑一下offlineimap，看看是否可以下载邮件了。offlineimap配置OK了，就可开始配置mu4e。


<a id="org0e3d134"></a>

## 查看邮件

首先是安装mu，mu中提供mu4e，[Github](https://github.com/djcb/mu)。
可参考配置:

    (add-to-list 'load-path "/usr/local/share/emacs/site-list/mu4e")
    (require 'mu4e)

    ;; 这部分配置要跟offlineimap一致，否则看不到任何邮件啊
    (setq
     mu4e-maildir "~/.mailbox"
     mu4e-sent-folder "/sent"
     mu4e-drafts-folder "/drafts"
     mu4e-trash-folder "/trash"
     mu4e-refile-folder "/archive"
     mu4e-attachment-dir  "~/Downloads"
     mu4e-html2text-command "html2text -utf8 -width 72")

    ;; 这是跳转文件夹的快捷键设置
    (setq mu4e-maildir-shortcuts
          '( ("/drafts"      . ?d)
             ("/sent"        . ?s)
             ("/trash"       . ?t)
             ("/flagged"     . ?f)
             ("/all"         . ?a)))

    ;; 这是HV的展示设置
    (setq mu4e-headers-fields
          '((:date          .  25)    ;m; alternatively, use :human-date
            (:flags         .   6)
            (:from          .  22)
            (:subject       .  nil))) ;; alternatively, use :thread-subject

    (setq mu4e-get-mail-command "offlineimap"
          mu4e-update-interval 300
          message-send-mail-function 'smtpmail-send-it)

emacs里执行mu4e就可以通过打开不通的文件查看邮件了。
由于墙的原因，用的免费vpn发不了邮件，所以"smtpmail-smtp-server"和"smtpmail-smtp-service"没有setq，一个是服务，另一个是端口。


<a id="orgb2eb34b"></a>

## Key Bindings


### Global

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left">C-c C-u / C-S-u</td>
<td class="org-left">Update email & database</td>
</tr>


<tr>
<td class="org-left">y</td>
<td class="org-left">switch between header view and message view</td>
</tr>


<tr>
<td class="org-left">s</td>
<td class="org-left">searching, [quering email](http://www.djcbsoftware.nl/code/mu/mu4e/Queries.html#Queries)</td>
</tr>
</tbody>
</table>


### Header View

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left">n / p</td>
<td class="org-left">go to next/previous header, and update message view if it's open</td>
</tr>


<tr>
<td class="org-left">] / [</td>
<td class="org-left">go to next/previous unread header, and update message view if it's open</td>
</tr>


<tr>
<td class="org-left">C-n / C-p</td>
<td class="org-left">go to next/previous header</td>
</tr>


<tr>
<td class="org-left">P</td>
<td class="org-left">toggle threading</td>
</tr>
</tbody>
</table>

[more](http://www.djcbsoftware.nl/code/mu/mu4e/Keybindings.html#Keybindings)


### Message View

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left">g</td>
<td class="org-left">go to numbered url</td>
</tr>


<tr>
<td class="org-left">k</td>
<td class="org-left">save the numbered url in the kill-ring</td>
</tr>


<tr>
<td class="org-left">o</td>
<td class="org-left">open numbered attachment</td>
</tr>


<tr>
<td class="org-left">e</td>
<td class="org-left">save numbered attachment</td>
</tr>
</tbody>
</table>

[more](http://www.djcbsoftware.nl/code/mu/mu4e/MSGV-Keybindings.html#MSGV-Keybindings)


### Mark for actions

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left">? / !</td>
<td class="org-left">mark message as unread/read</td>
</tr>


<tr>
<td class="org-left">d / D</td>
<td class="org-left">mark trash/delete</td>
</tr>


<tr>
<td class="org-left">%</td>
<td class="org-left">mark based on a regular expression</td>
</tr>


<tr>
<td class="org-left">T / t</td>
<td class="org-left">mark whole thread/subthread</td>
</tr>


<tr>
<td class="org-left">x</td>
<td class="org-left">execute actions for the marked messages</td>
</tr>
</tbody>
</table>

[more](http://www.djcbsoftware.nl/code/mu/mu4e/Keybindings.html#Keybindings)


<a id="org34c5331"></a>

# enjoying!

[mu4e manual](http://www.djcbsoftware.nl/code/mu/mu4e/index.html#SEC_Contents)
