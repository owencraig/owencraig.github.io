---
layout: post
title: Git bash aliases
permalink: /git-bash-aliases/
---
I like living in git bash on windows, however I also sometimes really need to jump into visual studio or sublime, so below are the two commands I run in git bash to add the shortcuts into bash.

####visual studio
{% highlight bash %}
echo "alias vs=\"/c/Program\ Files\ \(x86\)/Microsoft\ Visual\ Studio\ 12.0/Common7/IDE/devenv.exe\"" >> ~/.bashrc
{% endhighlight %}
_N.B. you may need to alter your path if you're running x86_

####sublime text 2
{% highlight bash %}
echo "alias subl=\"/c/Program\ Files/Sublime\ Text\ 2/sublime_text.exe\"" >> ~/.bashrc
{% endhighlight %}


Then I can just use
{% highlight bash %}

vs solution.sln &

{% endhighlight %}
or
{% highlight bash %}
subl file.md &
{% endhighlight %}