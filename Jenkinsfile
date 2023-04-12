pipeline{
agent  any
stages{
stage('test'){
    steps{
        script{
            echo "testing app at $BRANCH_NAME"
        }
        }
    }   
stage('build '){
    when{
        expression{
            BRANCH_NAME=='main'
        }
    }
    steps{
         script{
          echo 'buling app at $BRANCH_NAME' 
        }
    }
}
}
}
