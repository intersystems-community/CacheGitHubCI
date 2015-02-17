# CacheGitHubCI
Continuous Integration for InterSystems Caché and GitHub. 

Installation
-----------

1. Download zip and import classes into Caché (any namespace, further referred to as {Namespace}).

Webhook Installation
-----------

If you want to use webhooks functionality, additional installation steps are required.

1. Check that your server has static public IP address. 
2. Check that your server is not under NAT/Firewall/etc. by accessing Caché System Management Portal through external ip.
3. Set  ^CacheGitHubCI("IP") glopal (in namespace, where you imported classes) to the addpess of your server (with port, if required). For example: 

        set ^CacheGitHubCI("IP") = "45.45.45.45:57776"
        set ^CacheGitHubCI("IP") = "mycacheserver.com"
       
4. Create web application with unauthenticated access, named /cgci, with Dispatch Class = CacheGitHubCI.REST in {Namespace}

Usage 
-----------

There are two ways to create syncing between GitHub repository and Cache instance:

1. Simple Task
2. Hooks

Simple Task
-----------

To create task, syncing  GitHub repository -> Cache instance do the following:

1. Go to SMP → System Operation → Task Manager → New Task
2. Set <i>Name</i> and <i>Description</i> as desired
3. Set <i>Namespace to run task in</i> to {Namespace}
4. Set <i>Task type</i> to GitHubUpdateTask
5. Set <i>Branch</i> to required branch of repository
6. Set <i>GitHubURL</i> to a valid GitHub repository, eg: https://github.com/intersystems-ru/Cache-MDX2JSON
7. Set <i>Namespace</i> to a Namespace you want to download GitHub repository to
8. Set <i>Username</i> and <i>Password</i> to GitHub Username and Password. This is required only for private repositories OR if you plan to run the task often (unauthenticated users can query GitHub up to 60 times/hour)
9. Set other parameters as desired and finish creation of the task

After task runs at least once you will get <i>GitHubURL</i> repository contents in <i>Namespace</i>
