def project = 'android-project'

node {
    try {
        if(env.BRANCH_NAME == "beta" || env.BRANCH_NAME == "release"){
            notifyBuild('STARTED')
        }

        stage 'Checkout'
        deleteDir()
        checkout scm

        stage 'Create Docker image'
        sh("docker build -t ${project} .")

        stage('Build')
        switch (env.BRANCH_NAME) {
            case "master":
                sh("docker run --rm ${project} ./gradlew clean :app:assembleDebug")
                break
            case "beta":
                sh("docker run --rm ${project} ./gradlew clean :app:assembleBeta")
                break
            case "release":
                sh("docker run --rm ${project} ./gradlew clean :app:assembleRelease")
                break
        }

        if(env.BRANCH_NAME == "master") {
            stage('Unit Test')
            sh("docker run --rm ${project} ./gradlew jacocoTestDebugUnitTestReport checkstyle pmd")
        }

        if(env.BRANCH_NAME == "master") {
            stage('Integration Test')
            sh("docker run --rm ${project} ./android-emulator-control.sh start x86/24")
            sh("docker run --rm ${project} ./gradlew connectedDebugAndroidTest --info")
        }

        if(env.BRANCH_NAME == "beta" || env.BRANCH_NAME == "release") {
            stage 'Deploy'
            if(env.BRANCH_NAME == "beta"){
                sh("docker run --rm ${project} ./gradlew :app:crashlyticsUploadDistributionBeta")
            }
            if(env.BRANCH_NAME == "release"){
                sh("docker run --rm ${project} ./gradlew :app:publishApkRelease")
            }
        }
    } catch (e) {
        currentBuild.result = "FAILED"
        throw e
    } finally {
        notifyBuild(currentBuild.result)
    }
}

def notifyBuild(String buildStatus = 'STARTED') {
    buildStatus =  buildStatus ?: 'SUCCESSFUL'

    def color = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"
    def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""

    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFCC00'
    } else if (buildStatus == 'SUCCESSFUL') {
        color = 'GREEN'
        colorCode = '#228B22'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }

    slackSend (color: colorCode, message: summary, channel: "#android")
}
