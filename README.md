# Solution

This project consists of an Azure DevOps extension, containing one Azure DevOps Pipeline Decorator.
This decorator runs in every pipeline in the Azure DevOps organization where the extension is installed.
A decorator has a particular target; [Pipeline Decorator Targets](https://docs.microsoft.com/en-us/azure/devops/extend/develop/add-pipeline-decorator?view=azure-devops#targets)

This decorator only verifies the last commit on the active branch. Implying all other commits as valid and approved.

![image](https://user-images.githubusercontent.com/16139694/117464028-8d6e6980-af50-11eb-8bfc-57b883347a2e.png)

## Azure DevOps Requirements

To be able to use the pipeline to deploy the extension the Azure DevOps extension requires the [Azure DevOps Extension Tasks](https://marketplace.visualstudio.com/items?itemName=ms-devlabs.vsts-developer-tools-build-tasks) extension to be installed.

## Azure DevOps Publisher

Creating and publishing Azure DevOps extensions requires you to have a 'Publisher'. 
This publisher is the idenity/owner of the (private) Azure DevOps Marketplace items.
Read more on this here [Publisher](https://docs.microsoft.com/en-us/azure/devops/extend/publish/overview?view=azure-devops).

In the [azure-pipelines.yml](https://github.com/XpiritBV/azure-pipelines-verify-signed-decorator/blob/main/azure-pipelines.yml) we are using an extension to publish a freshly build Decorator towards an organization. In this case using JesseHouwing as publisher. 
 
The pipeline also holds the accounts it shares the exetension with. Typically in the Dev stage we target a non-production organization to test the deployment and extension behavior. If all is well we deploy to the production Azure DevOps organization with the Prod stage.

## Service Connection to Marketplace

To be able to automatically publish from the pipeline you need to setup a new Service Connection to the Azure DevOps Extension Marketplace.

![image](https://user-images.githubusercontent.com/16139694/117453255-05cf2d80-af45-11eb-9527-15ab9711b6d7.png)

## Authorize pipeline for using Service Connection

After this has been succesfully created you need to make sure the pipeline is authorized to execute the pipeline;
[Securing the Service Connection](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#secure-a-service-connection).

## Switches

The behavior of the decorator can be contrlld through 2 variables:

Variable | Behavior
------------ | -------------
`azure-pipelines-verify-signed-decorator.skip` | When in the Pipeline this variable has been set to `true` the decorator will be skipped.
`azure-pipelines-verify-signed-decorator.importKeys` | When in the Pipeline this variable has been set to `false` the decorator will not import the keys from the hardcoded location. This will allow you to manually import and control the keys in the keyring of a private agent.

Any other value will enable the default behavior.

## Public Key Storage

In order to identify your developers by their GPG Public Key these need to be made avaialble. 
We suggest creating an Azure Storage Account containing a blob container. 

There you specify a "keys" file. In the keys file can specify any public available URL for the GPG Public Key of the developer.
It is recommended to make each GPG Public key available in this blob container and specify the public URL in the keys file.
This blob container must be maintained by as limited amount of users as possible.
Use a tool like Azure Storage Explorer to maintain it conveniently.

![image](https://user-images.githubusercontent.com/16139694/117461575-04eec980-af4e-11eb-9540-7ad1ec9b5b97.png)

## Impact on Branching Strategy

There is no support for GPG signing of commits in Azure DevOps. Therefore any time Azure DevOps creater a commit on your behalf, it will not carry a valid signature. This will happen when you:

* Edit any file from the Web-UI directly
* Edit a YAML pipeline from the Web-UI directly
* Perform a pull-request that can't use the "fast-forward" strategy.
* Perform a merge, squash or rebase that results changes on the server.

A developer will have to pull the changes to a local repository, review them and then sign-off on these changes by signing off the commit.

To prevent a pull-request from generating an unsigned commit, set the "rebase and fast-forward" branch policy and always rebase or merge your branch locally prior to creating the pull-request. You may have to redo these steps everytime another pull-request is merged into the same branch.

![image](https://user-images.githubusercontent.com/16139694/117456985-1681a280-af49-11eb-8298-d3956e3aec82.png)

Any other merge-type policy will generate new commitsthat will not be signd and therefore won't pass validation.

[Merge Types documentation](https://docs.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-evops#limit-merge-types)

Any other merge-type policy will generate new commitsthat will not be signd and therefore won't pass validation.

Read more information on [branching strategies](https://docs.microsoft.com/en-us/azure/devops/repos/git/branch-policies-overview?view=azure-devops).
