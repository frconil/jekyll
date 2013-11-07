---
title: Quick syntax check for puppet manifests 
tags: Puppet Git 
layout: post
summary: Git is fast, but not so fast that you can test everything on your puppetmaster. Let's also admit that testing for syntax errors is bad form, and the more testing you do ahead of a commit, the better. 
---
Git is fast, but not so fast that you can test everything on your puppetmaster. Let's also admit that testing for syntax errors is bad form, and the more testing you do ahead of a commit, the better.

For the sole reason I just can't wait for something to be copied somewhere else, I have this quick pre-commit hook I wrote in bash.
It's not perfect and doesn't pick up ALL the mistakes, but it clears a lot of basic errors and typos by doing a basic flight-check.
Just copy the following in /path/to/puppet-repo/.git/hook/pre-commit

{% highlight  bash %}
#!/bin/bash
#
# basic puppet syntax check 
# 

for ppfile in `git diff --name-only HEAD~1 | grep "\.pp\|\.erb"` 
do
  if [[ -f $ppfile ]]
  then
    case $ppfile in
    *.pp)
      puppet parser validate $ppfile
      if [[ $? -ne 0 ]]
      then
        echo "$ppfile not validated by puppet parser"
        puppet_syntax=1
      else
        echo "$ppfile has been validated by puppet parser"
      fi ;;
    *.erb)
      erb -P -x -T '-' "$ppfile" | ruby -c
      if [[ $? -ne 0 ]]
      then
        echo "$ppfile not a valid erb file"
        puppet_syntax=1
      else
        echo "$ppfile is a valid erb file"
      fi ;;
    esac
  fi
done

exit $puppet_syntax;
{% endhighlight %}


