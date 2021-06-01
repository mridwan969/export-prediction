# export-prediction

**How to deploy Tensorflow API to Google Cloud Platform**

Login to Google Cloud Platform

1. Go to Navigation Menu > Compute Engine > VM Instances

    Create a VM instance to build a docker image:
    - choose e-2 standard-2 for the machine configuration
    - choose Debian GNU/Linux(Buster) for the operating system
    - set disk to Standard Persistent Disk and set the size to be 30GB

2. Activate Cloud Shell

    SSH to the VM instance that just created with the command:
    $ gcloud compute ssh _INSTANCE_NAME_ --zone _INSTANCE_ZONE_

3. Install Git and Git-LFS in order to be able to clone the dataset in the git repo in the VM with the commands below:
    $ sudo apt-get update
    $ sudo apt-get install git
    $ sudo apt-get update
    $ sudo apt-get install git-lfs

    Install Docker in the VM, just follow the steps in this article https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-debian-10

    After the installation done, clone the repository for the tensorflow api with the command:
    $ sudo git clone https://github.com/rahilwisdom/Projects.git

4. After cloning, go to the repo directory:
    $ cd Projects

    Build the docker image inside the directory:
    $ sudo docker build -t ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG .

5. Access this documentation https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry to learn how to work around github      PAT and how to use github container registry

6. After everything's done just push the image to github container registry by using the command:
   sudo docker push ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG

    When the push done, we can just delete the VM if there is no more use to it or you can just keep it to build another container.

    Exit the SSH by tiping exit

7. Go back to Google Cloud shell and login with your github account the same way as the fifth step.
8. Pull the image from github container registry to the cloud shell with the command:
    $ docker pull ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG
    
    then change the name of the image:
    $ docker tag ghcr.io/GITHUB_USERNAME/IMAGE_NAME:TAG asia.gcr.io/GCP_PROJECT_ID/IMAGE_NAME:TAG
    
    lastly, push the image to GCP container registry, in order to be safe we can just logout from github container registry by using the command:
    $ docker logout
    
    then push immediately
    $ docker push asia.gcr.io/GCP_PROJECT_ID/IMAGE_NAME:TAG
    
9. When the push is done, go to Navigation Menu > Container Registry > Images then select the newly pushed image.

   Click the three dots at the end of the image then choose which services do you want to deploy your image, in our case we decided to use Cloud Run since we can use continous      deployment for our API.
   
10. Deploy to Cloud Run, then select your desired region, leave the rest to default and then deploy.

11. In addition you can add your image to your github repositories by navigating to github in the top left, go to Your Profile > Packages
    Click the newly pushed image then connect it to your repository.
    Go to repository and confirm that the image has been connected to the repository.
    

That's all the steps for deploying the Tensorflow API.



