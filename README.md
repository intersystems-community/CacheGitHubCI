# CacheGitHubCI
Continuous Integration for InterSystems Caché and GitHub. 

Installation
-----------

1. Import Install.cls.xml (from  folder in repository or releases page) into Caché via Studio or Terminal or SMP
2. Run in terminal (any namespace): 

        Set pVars("Namespace") = "CGCI" 
        Set pVars("IP") = {IP} 
        Set pVars("Branch") = {Branch}
        Do ##class(CacheGitHubCI.Install).setup(.pVars)

  where: 
  
      {Namespace} is a namespace you want to install to. If it does not exist it would be created automatically. If it does exist only CacheGitHubCI package would be overwritten. 
	  {IP} is an optional parameter of address of your server (with port, if required). For example "45.45.45.45:57776", "mycacheserver.com". Required if you wish to use webhook functionality.
	  {Branch} is an optional parameter of branch from which you want to download CacheGitHubCI (only used while an online installation). If omitted master branch will be used.
On this step installer would create (if needed) Namespace and corresponding database, download source code from GitHub and compile it, optionally create web application (named /cgci) if one does not exist (skipping web application creation process if one does exist or no IP is provided).

3. Give the correct roles to `/cgci` webapplication for it to be able to compile desired classes.
4. Check that `^CacheGitHubCI("IP")` global is set to {IP}. Set manually, if not.


Webhook Installation
-----------

If you want to use webhooks functionality, additional installation steps are required.

1. Check that your server has static public {IP} address. 
2. Check that your server is not under NAT/Firewall/etc. by accessing Caché System Management Portal through {IP}.

Usage 
-----------

There are two ways to create syncing between GitHub repository and Cache instance:

1. Simple Task
2. Hook

Simple Task
-----------

To create task, syncing  GitHub repository → Cache instance do the following:

1. Go to SMP → System Operation → Task Manager → New Task
2. Set <i>Name</i> as desired
3. Set <i>Namespace to run task in</i> to {Namespace}
4. Set <i>Task type</i> to GitHubUpdateTask
5. Set <i>GitHubURL</i> to a valid GitHub repository, eg: https://github.com/intersystems-ru/Cache-MDX2JSON
7. Set <i>Namespace</i> to a Namespace you want to download GitHub repository to
8. Set other parameters as desired and finish creation of the task

After task runs at least once you will get <i>GitHubURL</i> repository contents in <i>Namespace</i>

Hook
-----------

To create more sophisticated setups you need to use CacheGitHubCI.Hook. Usage example:

    Set hook=##class(CacheGitHubCI.Hook).%New()     // Create hook
    Set hook.Namespace="user"                       // Set namespace you want to download GitHub repository to
    Set hook.Owner="intersystems-ru"                // Set repository owner
    Set hook.Repository="cache-mdx2json"            // Set repository name
    Set hook.Branch="master"                        // Set repository branch
    Do hook.%Save()
    
In this example we created a hook to download [MDX2JSON](https://github.com/intersystems-ru/Cache-MDX2JSON) repository into USER namespace. Now we activate it by creating a task or a webhook to update it. Task updates namespace with repository contents every X minutes (60 by default):

    W hook.CreateTask(60)                           // Create task to run every 60 minutes
    Do hook.%Save()
        
Webhook updates repository only when someone pushes changes into it. To use webhook functionality you must complete steps, described in Webhook Installation part of this document. To create webhook do:

    Set hook.Username="GitHub Username"           // Required for private repositories or if you want to use webhooks
    Set hook.Password="GitHub Password" 
    W hook.CreateHook()                           // Creates GitHub webhook
    Do hook.%Save()
        
Hook execution
-----------

Every time hook gets activated (by webhook or task) it executes the following series of steps: PreCompile → Compile → PostCompile → UnitTests. You can (optionally) supply the code for PreCompile, PostCompile, UnitTests steps and results of their execution get recorded in CacheGitHubCI.Update object. Here's how to create actions:

    Set a1 = ##class(CacheGitHubCI.Action).%New()   // Create new action
    Set a1.Type="code"                              // Set it as COS code
    Set a1.Params="s ^test($zdt($Now(-180)))=""started compiling""" // Set what COS code to execute 
    Set hook.PreCompile=a1                          // Set it to execute every time before downloading and compiling repository
        
    Set a2 = ##class(CacheGitHubCI.Action).%New()
    Set a2.Type="classmethod"                       // Set it as classmethod
    Set a2.Namespace="USER"                         // Set namespace to run action as USER ({Namespace} by default)
    Set a2.Params="MDX2JSON.REST,Test"              // classmethod,classname,arg1,...,argN
    Set hook.PostCompile=a2                         // Set it to execute every time after downloading and compiling repository
        
    Set a3 = ##class(CacheGitHubCI.Action).%New()
    Set a3.Type="classmethod"
    Set a3.Namespace="USER"
    Set a3.Params="MDX2JSON.REST,Test"
    Set hook.UnitTests=a3                           // Action For UnitTest
    
    Do hook.%Save()

Updates History
-----------
To see the history of updates you can execute following SQL query in {Namespace}:

    SELECT * FROM CacheGitHubCI."Update" WHERE Hook = 'owner||repository||namespace'
        
So, for Cache-MDX2JSON the request would look like:

    SELECT * FROM CacheGitHubCI."Update" WHERE Hook = 'intersystems-ru||cache-mdx2json||user'
	
To see what’s going on with all your hooks:

    SELECT Namespace, Owner, Repository, LastUpdateDateTime, LastUpdateStatus FROM CacheGitHubCI.Hook

LastUpdateStatus  is result of latest update attempt:

    Successful - update successful
    Reverted - update error, reverted to last successful commit 
	
DeepSee
-----------

There is DeepSee cube CacheGitHubCICube in {Namespace}. It updates automatically after every hook update. There is also a pivot and a dashboard providing information about available hooks and their status. 

