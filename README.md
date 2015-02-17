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