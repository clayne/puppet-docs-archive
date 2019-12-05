# How Code Manager works

Code Manager uses r10k and the file sync service to stage, commit, and sync your code, automatically managing your environments and modules.

First, create a control repository with branches for each environment that you want to create \(such as production, development, or testing\). Create a Puppetfile for each of your environments, specifying exactly which modules to install in each environment. This allows Code Manager to create directory environments, based on the branches you've set up. When you push code to your control repo, you trigger Code Manager to pull that new code into a staging code directory \(`/etc/puppetlabs/code-staging`\). File sync then picks up those changes, pauses Puppet Server to avoid conflicts, and then syncs the new code to the live code directories on your masters.

For more information about using environments in Puppet, see [About Environments](https://puppet.com/docs/puppet/5.3/environments_about.html).

## Understanding file sync and the staging directory

To sync your code across multiple masters and to make sure that code stays consistent, Code Manager relies on file sync and two different code directories: the staging directory and the live code directory.

Without Code Manager or file sync, Puppet code lives in the codedir, or live code directory, at `/etc/puppetlabs/code`. But the file sync service looks for code in a code staging directory \(`/etc/puppetlabs/code-staging`\), and then syncs that to the live codedir.

Code Manager moves new code from source control into the staging directory, and then file sync moves it into the live code directory. This means you no longer write code to the codedir; if you do, the next time Code Manager deploys code from source control, it overwrites your changes.

For more detailed information about how file sync works, see the related topic about file sync.

CAUTION:

Don't directly modify code in the staging directory. Code Manager overwrites it with updates from the control repo. Similarly, don't modify code in the live code directory, because file sync overwrites that with code from the staging directory.

**Related information**  


[About file sync](filesync_about.md#)

## Environment isolation metadata and Code Manager

Both your live and staging code directories contain metadata files that are generated by file sync to provide environment isolation for your resource types.

These files, which have a `.pp` extension, ensure that each environment uses the correct version of the resource type. Do not delete or modify these files. Do not use expressions from these files in regular manifests.

These files are generated as Code Manager deploys new code in your environments. If you are new to Code Manager, these files are generated when you first deploy your environments. If you already use Code Manager, the files are generated as you make and deploy changes to your existing environments.

For more details about these files and how they isolate resource types in multiple environments, see environment isolation.

## Moving from r10k to Code Manager

Switching from r10k to Code Manager can improve automation of your code management and deployments, but some r10k users might not be ready to switch yet.

Code Manager uses r10k in the background to improve automation of your code management and deployment. However, switching to Code Manager means you can no longer use r10k manually. Because of this, some r10k users might not want to move to Code Manager yet. Specifically, be aware of the following restrictions:

-   If you use Code Manager, you **cannot** deploy code manually with r10k. If you depend on the ability to deploy modules directly from r10k, using the `r10k deploy module` command, you should continue to use your current r10k workflow.
-   Code Manager must deploy all control repos to the same directory. If you are using r10k to deploy control repos to different directories, you should continue to use your current r10k workflow.
-   Code Manager does not support system .SSH configuration or other shellgit options.
-   Code Manager does not support post-deploy scripts.

If you rely on any of the above configurations, or any other r10k configuration that Code Manager doesn't yet support, you should continue to use your current r10k workflow. If you are using a custom script to deploy code, you should carefully assess whether Code Manager meets your needs.

**Related information**  


[Managing code with r10k](r10k.md)
