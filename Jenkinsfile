@Library(['maas-jenkins-library@master']) _


import com.cloudbees.hudson.plugins.folder.Folder
import jenkins.branch.OrganizationFolder
import org.jenkinsci.plugins.workflow.job.WorkflowJob
import org.jenkinsci.plugins.workflow.multibranch.WorkflowMultiBranchProject
import hudson.model.Item


pipeline {
    agent {
        kubernetes {
            label "delete-old-builds-${UUID.randomUUID().toString().substring(0, 8)}"
            yaml """
      apiVersion: v1 
      kind: Pod 
      spec:
        containers: 
          - name: jenkins-slave
            image: jenkins/jnlp-slave:3.35-5-alpine
            imagePullPolicy: Always
            command: ['cat']
            tty: true
      """
        }
    }

    triggers { cron('0 3 * * *') }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5', daysToKeepStr: '5'))
    }

    stages {
        stage('Clean Old Builds') {
            steps {
                script {
                    for (item in Jenkins.instance.items) {
                        if (item.getName() == 'build'){
                            echo "Clean Builds Only"
                            deleteOldBuilds(item)
                        }
                    }
                }
            }
        }
    }
}

void deleteOldBuilds(Item item) {
    if(item instanceof Project) {
        echo "PROJECT:(${item.getName()})"
    } else if(item instanceof Folder) {

        echo "FOLDER: (${item.getName()})"
        for (subItem in item.items) {
            deleteOldBuilds(subItem)
        }
    } else if(item instanceof WorkflowMultiBranchProject) {
        echo "MULTIBRANCH-PROJECT: ${item.getName()}"
        for (subItem in item.items) {
            deleteOldBuilds(subItem)
        }
    }  else if(item instanceof WorkflowJob) {
        if (!item.isBuildable()){
            echo "MULTIBRANCH-JOB: (${item.getName()} Buildable: ${item.isBuildable()}"
            //item.delete()
        }
    } else if(item instanceof OrganizationFolder) {
        echo "ORG-FOLDER: ${item.getName()}"
        for (subItem in item.items) {
            deleteOldBuilds(subItem)
        }
    } else {
        echo "UNKNOWN: ${item.getName()}"
        echo "CLASS: ${item.getClass()}"
        echo "INSPECT: ${item.inspect()}"
    }
}

