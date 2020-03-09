# Create Branches and Perform Operations
## Branching
Branches help you to work on different versions of a repository at one time. Let’s say you want to add a new feature (which is in the development phase), and you are afraid at the same time whether to make changes to your main project or not. This is where git branching comes to rescue. Branches allow you to move back and forth between the different states/versions of a project. In the above scenario, you can create a new branch and test the new feature without affecting the main branch. Once you are done with it, you can merge the changes from new branch to the main branch. Here the main branch is the master branch, which is there in your repository by default.  

Refer to the below image for better understanding:
![Branching - how to use GitHub](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2017/11/Branching-how-to-use-GitHub-Edureka-1.png)  

As depicted in the above image, there is a master/ production branch which has a new branch for testing. Under this branch, two set of changes are done and once it completed, it is merged back to the master branch. So this is how branching works!  

To create a branch in GitHub, follow the below steps:
* Click on the dropdown “Branch: master”
* As soon as you click on the branch, you can find an existing branch or you can create a new one. In my case, I am creating a new branch with a name “readme- changes”. Refer to the below screenshot for better understanding.  

![CreateBranches - how to use GitHub](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2017/11/CreateBranches-how-to-use-GitHub-Edureka.png)  

Once you have created a new branch, you have two branches in your repository now i.e. read-me (master branch) and readme- changes. The new branch is just the copy of master branch. So let’s perform some changes in our new branch and make it look different from the master branch.  

## GitHub Operations
### Commit Command
This operation helps you to save the changes in your file. When you commit a file, you should always provide the message, just to keep in the mind the changes done by you. Though this message is not compulsory but it is always recommended so that it can differentiate the various versions or commits you have done so far to your repository. These commit messages maintain the history of changes which in turn help other contributors to understand the file better.  

Now let’s make our first commit, follow the below steps:  
* Click on “readme- changes” file which we have just created.
* Click on the “edit” or a pencil icon in the righmost corner of the file.
* Once you click on that, an editor will open where you can type in the changes or anything.
* Write a commit message which identifies your changes.
* Click commit changes in the end.

![Commit - how to use github](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2017/11/Commit-how-to-use-github-Edureka.png)

We have successfully made our first commit. Now this “readme- changes” file is different from the master branch. 

### Pull Command
Pull command is the most important command in GitHub. It tell the changes done in the file and request other contributors to view it as well as merge it with the master branch. Once the commit is done, anyone can pull the file and can start a discussion over it. Once its all done, you can merge the file. Pull command compares the changes which are done in the file and if there are any conflicts, you can manually resolve it.  

Now let us see different steps involved to pull request in GitHub.  
* Click the ‘Pull requests’ tab.
* Click ‘New pull request’.
* Once you click on pull request, select the branch and click ‘readme- changes’ file to view changes between the two files present in our repository.
* Click “Create pull request”.
* Enter any title, description to your changes and click on “Create pull request”. Refer to the below screenshots.

![Pull - how to use github 01](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2017/11/Pull-commands-how-to-use-github-Edureka.png)  
![Pull - how to use github 02](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2017/11/Pull-command-how-to-use-github-Edureka.png)  

### Merge Command
Here comes the last command which merge the changes into the main master branch. We saw the changes in pink and green color, now let’s merge the “readme- changes” file with the master branch/ read-me.  

Go through the below steps to merge pull request.
* Click on “Merge pull request” to merge the changes into master branch.
* Click “Confirm merge”.
* You can delete the branch once all the changes have been incorporated and if there are no conflicts. Refer to the below screenshots.  

![Merge command - how to use github](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2017/11/Merge-command-how-to-use-github-Edureka-1.png)
