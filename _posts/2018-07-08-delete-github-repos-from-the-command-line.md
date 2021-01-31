---
title: Delete github repos from the command line
category:
- github
date: '2016-05-01 15:23:48'
---

Adapted from: https://medium.com/@icanhazedit/clean-up-unused-github-rpositories-c2549294ee45#.1yh9lzg1p

My github profile was a hot mess: I had many repos I forked during coding bootcamp and never developed on. I decided to learn a bit of bash command line and remove these sad, undeveloped repos from my github account.

Here is how you can do the same:

1. Install this nifty command line JSON parser and its dependencies. 
	Note: I used jsawk to get the names of my github repositories from the large JSON response returned by Github's API instead of collecting the list manually. If there is a simpler way to do this without installing these programs, let me know!

	```
	$ brew install jsawk && brew link jsawk
	$ brew install spidermonkey && brew link spidermonkey
	```

2. Get a list of all the repositories you own and save this to a repos.txt file using the Github API: https://developer.github.com/v3/repos/
```bash
$ curl https://api.github.com/users/ugotsoul/repos?type=owner | jsawk -n 'out(this.full_name)' > repos.txt
```
3. Register a new personal access token with the delete_repo permmission: https://github.com/settings/tokens/new
4. Copy the access token and save this to your bash profile:
```
$ echo "export GITHUB_TOKEN=<TOKEN>" >> ~/.bash_profile
$ source ~/.bash_profile
$ echo $GITHUB_TOKEN
$ … should output token value …
```
5. Inspect the repos.txt file and remove the repositories you want to keep.
**WARNING**: Make sure you have double-checked your list of repos you want to delete before running any of these commands listed below.
6. Let's be safe: only delete oneitem/repository from the list:
```bash
$ head -n 1 repos.txt | xargs -I % curl -X "DELETE" "https://api.github.com/repos/"%"?access_token=$GITHUB_TOKEN"
```
7. Run the above command again. If the DELETE was successful, you should have a response like:
```
{
"message": "Not Found",
"documentation_url": "https://developer.github.com/v3"
}
```
8. Now, run the command below to delete all the repositories in the list.
```bash
$ cat repos.txt | xargs -I % curl -X "DELETE" "https://api.github.com/repos/"%"?access_token=$GITHUB_TOKEN
```
Wait a sec: what's `xargs` and what does it do?
> [Manual page] xargs reads standard input (the stream data going into a program, eg: the text input from your keyboard), delimited by spaces or newlines, one or more times with any initial arguments followed by the standard input provided.

Let's break this down:
By default, xargs executes echo as the initial command argument which redirects standard input to standard output and into your terminal:
```
cat repos.txt | xargs
ugotsoul/Ex2 ugotsoul/ex6 ugotsoul/ex8 ugotsoul/google-api-ruby-client ugotsoul/HBEx4 ugotsoul/HBSkills1 ugotsoul/hmwk2 ugotsoul/PhotoBOO ugotsoul/rails_tutorial ugotsoul/skills3 ugotsoul/stellar.js ugotsoul/W1-Repeats
```
I want to use xargs to execute a command which requires string substitution. I can pass a `-I` flag along with a placeholder `%` to xargs to represent each line of the list of repos as the % placeholder, and then pass a command while doing string substitution on the list item:
```
~/personal cat repos.txt | xargs -I % echo 'test' % 'this works?'
test ugotsoul/Ex2 this works?
test ugotsoul/ex6 this works?
test ugotsoul/ex8 this works?
test ugotsoul/google-api-ruby-client this works?
test ugotsoul/HBEx4 this works?
test ugotsoul/HBSkills1 this works?
test ugotsoul/hmwk2 this works?
test ugotsoul/PhotoBOO this works?
test ugotsoul/rails_tutorial this works?
test ugotsoul/skills3 this works?
test ugotsoul/stellar.js this works?
test ugotsoul/W1-Repeats this works?
```

**Now the final verification step:**

View the remaining repos on your account: I ended up deleting over 15 unused repositories (!) and now have a grand total of 10 repos on my github account.
```bash
curl https://api.github.com/users/ugotsoul/repos?type=owner | jsawk -n 'out(this.full_name)'
```

I hope this was helpful! Please [let me know](mailto:ugaso@learningishard.af?subject="{{ page.title }}") of any errors or omissions.