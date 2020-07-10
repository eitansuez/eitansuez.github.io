2020.07.09

# Preparing for the CKAD certification exam

In late June 2020, I took (and passed) the [Certified Kubernetes Application Developer (CKAD) exam](https://www.cncf.io/certification/ckad/).

In retrospect, what can I share with you that might be useful, assuming your goal is to prepare, take and pass this same certification examination?

At a high level, the main concepts one must understand are:

- Pods
- Labels and selectors
- Replicasets
- Deployments
- Services (mainly clusterip and nodeport)
- Volumes, PersistentVolumes, PersistentVolumeClaims
- ConfigMaps and Secrets
- Ingress controllers and resources
- Network policies
- Jobs and CronJobs
- Taints and tolerations
- Node selectors and node affinity

Regarding configmaps and secrets, this includes how to set an environment variable from a secret or configmap, and this includes both the `envFrom:` and `valueFrom:` variants.  Further, secrets and configmaps are valid types of volumes. So know how to configure both secrets and configmaps as volumes.

Regarding pods, you should know:

- the basics: creating a pod with single container using a specific image
- how to configure liveness and readiness probes
- how to set resource requests and limits
- configuring volume mounts, environment variables
- how to configure pods with multiple containers, and further, understanding how multiple containers within a pod can communicate to one another (for example, how one can produce a file that is consumed by the other).
- how to configure the security context at the pod or container level, including container capabilities

Regarding volumes, you should be aware of and able to configure different types, not just configmaps and secrets, but also `emptyDir`, `nfs`, and `hostPath`. There are other details about volumes you should get comfortable with, including storage classes.

But the list goes on:
 
- how to deploy an ingress controller, how to configure an ingress resource
- jobs and cronjobs
- all about deployments.  how to create them, edit them, set the rollout
  strategy, rollout an upgrade, or rollback to a previous version
- setting network policies to allow or forbid access (ingress/egress) to and from certain pods.

And therein lies the difficulty of the exam. There's just a lot to it.  Luckily the list doesn't go on forever, it is finite.

With practice, it does all sink in.  And that's primarily what I advise you do: practice a great deal over a compressed period of time.  It's uncanny how much better you get over a short period of time.  Set aside the time and give yourself a deadline.  I gave myself a week.  I started on a Monday. On Friday I scheduled the exam for the following Monday, and spent the (entire) intervening weekend practicing.  By Monday I was ready.  I made sure I slept well the night before the exam.

Honestly I'm surprised that, not having touched k8s in over a week since taking the exam, I remember this much. :-)

One dimension of learning has to do with knowing how to verify that whatever you were instructed to create has been created, is in a good state, and is functioning as you expect.  Sometimes verifying that something functions involves having to obtain a shell prompt inside
a running pod and poking around in there.

A good exercise I recommend as you prepare, is:

Each time you are challenged with a kubernetes task (whether it's to create a pod, a deployment, expose a service, etc..), don't stop there:  figure out how to verify that you've done it right, what commands can help you see state information?  It could be something
as simple as `kubectl get pods --show-labels` or making use of the `-o wide` flag, or just listing your volume claims to see whether they're bound to the persitent volume you previously created.

Sometimes a brief listing with a few extra columns is a simpler to sift through compared to the output of the `kubectl describe` command.

But more importantly, asking yourself and learning in each situation exactly how to get at the information you need is not only useful, but necessary while taking the exam, both for analysis and troubleshooting purposes, and to verify that you completed the task you were given correctly.

You should know how to easily create yourself a pod for the sake of obtaining a shell prompt inside the cluster.  Make sure this becomes second nature.

The [Udemy course by Mumshad Mannambeth](https://www.udemy.com/course/certified-kubernetes-application-developer/) is designed to prepare you for the test. This was for me (and many others) the principal preparation tool.  Kudos to Mumshad and all involved in the creation of this course.

My advice is: work through the course, then:

- Rework the exercises until concepts and tasks become second nature.
- Rework the lightning labs until your time to complete all exercises gets below that 30 minute threshold.
- Rework the practice tests until your comfort level is all the way up to "confident".

Don't stop there: after conquering the Udemy course, you will likely not be "there" yet (you may be if you use k8s daily in your work, but this was not the case for me).

Set aside time to collect external resources: blog entries, github repositories with information, tips, and resources.

Here's [a blog entry](https://codeburst.io/the-ckad-browser-terminal-10fab2e8122e) that I found helpful.
Don't just read the blogs: exercise the advice, the tips you are given, incorporate them into your toolbox.
People have done remarkable things in setting up resources for studying for this exam.
For example, here's a [great resource](https://github.com/dgkanatsios/CKAD-exercises/) I came across in the course of doing [some of](https://medium.com/faun/ckad-exam-tips-6e00cc56b6a1) that homework.

Something interesting happens over time. You get better at things. You don't just get the answer right, the familiarity increases to a point where things become second nature.

Each time something behaves in a way you do not predict is an opportunity to dig deeper and understand why, and gain a deeper understanding of how things work.  Do this over and over again until there are no mysteries left.

Play, tinker a lot.

[Setup minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) on your local machine so you can try things out ad-hoc.
Or use your public cloud account to spin up a cluster for yourself.
It's wonderful and convenient to use the Udemy course practice tests as a means to get access to a k8s environment.
But it's much easier to have one at your disposal to try things out whenever you like.
It's all about building up your fluency.

Regarding test taking, pay attention to:

- Typos will kill an otherwise perfect score.  If you're instructed to create a pod with a name such as `my-pod-xyzsasdf33`, it's easy to get that wrong. The test environment helps you avoid such mistakes by providing the ability to copy and paste names directly from the problem description.  Use copy/paste as a means to avoid making typos.

- When you finish implementing a task, get in the habit to not just go on to the next question. Take an extra 1-2 minutes to review precisely the names of the resources you've produced in your k8s cluster and validate them against the names from the problem description.

- Speed.  You have to be fast.  Many small habits can add up to saving a lot of time. Practicing, repeating the construction of commands will make you faster. Knowing _a prioris_ the outcome of the command you know to invoke is ultimately where you want to be.

Here are some speed-related tips.

## Aliases

`k` is easier and faster to type than `kubectl`.

But there's a catch:  in my experience, command completion did not always work when using `k` as a substitute for `kubectl`.

I also noticed that the choice of key/value separator (a space vs an equal sign) impacted command completion.

For example, this works:

`kubectl create deployment --image nginx --<tab>`
 
but not this:

`kubectl create deployment --image=nginx --<tab>`

Command completion is important to me not only for speed but also as a checksum that I am using my command correctly.
Having said that, you should be prepared to complete the exam even without the aid of command completion.
When I started my test, it wasn't enabled.  I turned it on using the [`kubectl completion`](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enable-kubectl-autocompletion) command.

An alias can make short work of switching namespaces.

I didn't go crazy with aliases. This is all I used:

```bash
alias k=kubectl
alias kns="kubectl config set-context --current --namespace"
export DR="--dry-run=client -oyaml"
```

Use `kns` by appending the name of the namespace you wish to switch to.

The _DR_ (stands for _Dry Run_) environment variable above sure beats having to type `--dry-run=client -oyaml` each and every time. Place the above into your `.bashrc` and source it at the beginning of your exam.  This is one of many tips I adopted after reading advice out in the blogosphere.

Here's an example command that first sets the namespace to _my-ns_ and then writes a deployment manifest to the console:

```bash
kns my-ns
k create deploy my-web-server --image nginx $DR
```

I discovered these and other shortcuts and suggestions while reading others' blogs on the subject, all the credit is theirs.

## Tips

- If I wanted to generate yaml with an imperative command, I'd often run it without output redirection just to verify that the command worked.  Once I see it works, I re-invoke the command with a `> my-resource.yml` appended.

- I use vim a lot.  So I `set -o vi`, which allows me to edit away a typo in a long command instead of having to repeat the command over again, or be restricted to using arrow keys for navigation.

- Using imperative commands can take all of the drudgery out of writing yaml by hand.  Learn to `kubectl create`, `kubectl run`, and `kubectl expose`.

- One issue to be aware of, especially if like me, you have not been working with k8s for long, has to do with changes to the way the kubectl cli works in v1.18.  Specifically the "imperative commands." `kubectl run` used to be polymorphic but now creates only pods.  `kubectl create` is the "replacement" for some of the things that `kubectl run` used to do.

    Here's a [blog entry](https://medium.com/faun/be-fast-with-kubectl-1-18-ckad-cka-31be00acc443) that does a great job of summarizing the changes.

- Learn how to get to a template yaml quickly from the reference documentation.  For example, search the [k8s docs](https://kubernetes.io/docs/home/) for "persistentvolume", select the first search result, then _Find in page_ for  "kind:" can quickly get you to the snippet you need.


## Aside: about snippets

On my mac, I use [alfred](https://www.alfredapp.com/) as a command launcher, but alfred also sports a clipboard history and a snippets feature.  Using alfred's snippets to paste yaml would shave even more time out of the tedium of looking up a template, but I chose not to use it, so as not to risk having that considered in some way cheating.

## More Tips

- At every opportunity learn and switch to using the short name of every k8s api resource (see `kubectl api-resources`). That's `ns` for namespace, `pv` for persistent volume, `cj` for cronjob, `netpol` for network policy, and so on.

- [Learn the kubectl cli thoroughly](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands).  Familiarize yourself with even the less often used commands, such as `kubectl get events`.

- Set your `.vimrc` file. Here is the one I used, albeit it may not be ideal:

    ```vimrc
    set tabstop=2 softtabstop=2 shiftwidth=2
    set number expandtab ruler autoindent smartindent
    syntax enable
    filetype plugin indent on
    ```

    With vim, I've run into issues with pasting with indent mode turned on (on linux, but not on my mac, perhaps due to a version difference). It doesn't hurt to know to `:set paste` to get around that.  Better yet, you can configure a [paste toggle](https://vim.fandom.com/wiki/Toggle_auto-indenting_for_code_paste).

    Also ":retab" is definitely good to know just in case an errant tab makes its way into your yaml file. No need to search for tabs in your yaml.

- I love `.inputrc` for history search (see [the single most useful thing in bash](https://coderwall.com/p/oqtj8w/the-single-most-useful-thing-in-bash)).  I set it first thing as the exam begins, and I do a `bind -f ~/.inputrc` to have it take effect.

- About namespaces:  be careful to check what your current namespace is set to before you start creating resources in your k8s cluster.
Get in a habit to reset it at the end of an exercise or at the beginning of each new problem.

- `kubectl explain` is important as a means of learning the yaml vocabulary.  Also, if for some reason during the test you need quick access to the names of the fields you can set on a specific section of yaml, this command comes in handy.  Here's a simple example:

    ```bash
    kubectl explain pod.spec.containers.securityContext --recursive
    ```

One more thing about your ability to complete an exercise at a fast pace:  the faster you get at this, the less pressure you'll be under when you take the test.  You'll have time to check your work, go back to that difficult question and get that one right too.

Good luck, and I hope the above advice helps you.

