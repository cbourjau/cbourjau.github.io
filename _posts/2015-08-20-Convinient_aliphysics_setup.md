---
layout: post
title:  "Making a nice and clean copy of the aliphysics repository"
categories: alice aliroot aliphysics
---

I don't see the point in using this git workbench thingy. For me it just gets in the way and I rather stick to the standard, plain git. The following is how I would think aliphysics development should be done, but correct me if I'm wrong! I am not a git expert! First off, I created my own aliphysics repository in order to have commit rights. This gives me the warm and fuzzy feeling of always having my code backed. Plus I can commit as often as I like without annoying anyone. I recommend setting up an aliphysics clone at `http://gitlab.cern.ch`. Create a new project there, by cloning the "upstream" aliphysics repository `http://git.cern.ch/pub/AliPhysics`. Once this is done, make a local clone on your computer. I followed the folder structure from Dario's tutorial ([link](https://dberzano.github.io/alice/install-aliroot/manual/#aliphysics)) in order to be still able to use his source file for setting up the environment. In other words: I used his automatic installer and then just replaced the "master" folder in the aliphysics directory with my own clone (altering the url, of cause):


{% highlight ruby %}
~/alicesw/aliphysics$ rm -r master
~/alicesw/aliphysics$ git clone ssh://git@gitlab.cern.ch:7999/cbourjau/aliphysics.git master
{% endhighlight %}


Now, your local repository (called "origin") is linked up with your own online aliphysics repository to which you have commit rights.

Now, you can develop in this repository to you liking and just push whenever you like to backup your process! Once everything is working, anyone with commit rights to the upstream aliphysics can just pull your changes from your repository and merge them into the original one.

Furthermore, gitlab has nice reviewing tools like commenting on individual lines etc. which make collaborating munch easier.

In order to sync the origin with the changes in the upstream repository, one first hast to add the url for the upstream repository:

{% highlight ruby %}
$ cd master
$ git remote add upstream <http://git.cern.ch/pub/AliPhysics>
{% endhighlight %}


Then, fetch all the branches and tags from that remote (read: remote repository):

{% highlight ruby %}
$ git fetch -p upstream
{% endhighlight %}


Then make sure you are on the master branch of your local repository (or wherever you want to merge the upstream changes into) and run the merge (meaning "merge upstream/master" into the current local branch):

{% highlight ruby %}
$ git checkout master
$ git merge upstream/master
{% endhighlight %}


Last but not least, push that merge into your own remote repository:

<div class="sh">
$ git push origin master

</div>

