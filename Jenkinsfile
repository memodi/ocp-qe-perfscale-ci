
pipeline {
    environment {
        // environment variable declared here will retain its values  
        // through all stages when accessesed as ${env.VAR} 
        // If the env var values are updated during the stages it should be accessed as ${VAR}
        NOO_BUNDLE_VERSION = null
    }
    agent any
    stages {
        stage("Capture Workload results") {
            steps {
                script {
                    captureWorkloadResults()
                }
            }
        }
    }

}

def getTimestamp(isodate) {
    def command="date -d ${isodate} +%s"
    def timestamp = sh(script: command, returnStdout: true).trim()
    return timestamp
}

def captureWorkloadResults(){
    def UUID = ''
    def JENKINS_BUILD = ''

   

        env.KUBEBURNER_BUILD = 2195

        // copy artifacts from kube-burner-workload job
        copyArtifacts fingerprintArtifacts: true, projectName: "scale-ci/e2e-benchmarking-multibranch-pipeline/kube-burner-ocp", selector: specific(env.KUBEBURNER_BUILD), target: 'kube-burner-workload-artifacts', flatten: true

        kubeburnerWorkloadInfo = readJSON(file: "$WORKSPACE/kube-burner-workload-artifacts/index_data.json")
        kubeburnerStarttime = getTimestamp(kubeburnerWorkloadInfo.startDate)
        kubeburnerEndtime = getTimestamp(kubeburnerWorkloadInfo.endDate)
        UUID=kubeburnerWorkloadInfo.uuid

         env.INGRESSPERF_BUILD = 591
        // copy artifacts from ingress-perf job
        copyArtifacts fingerprintArtifacts: true, projectName: "scale-ci/e2e-benchmarking-multibranch-pipeline/ingress-perf", selector: specific(env.INGRESSPERF_BUILD), target: 'ingressperf-workload-artifacts', flatten: true

        ingressperfWorkloadInfo = readJSON(file: "$WORKSPACE/ingressperf-workload-artifacts/index_data.json")
        ingressperfStarttime = getTimestamp(ingressperfWorkloadInfo.startDate)
        ingressperfEndtime = getTimestamp(ingressperfWorkloadInfo.endDate)
        if (params.WORKLOAD == 'ingress-perf'){
            UUID=ingressperfWorkloadInfo.uuid
            JENKINS_BUILD = "${ingressperfWorkloadInfo.getNumber()}"
        }
    
    // evaluate workload start time and end times
    def START_TIME = ''
    def END_TIME = ''
    if (params.RUN_INGRESS_PERF_IN_PARALLEL) {
        // use earliest start time
        if (ingressperfStarttime < kubeburnerStarttime) {
            START_TIME = ingressperfStarttime
        }
        else {
            START_TIME = kubeburnerStarttime
        }
        // use latest end time
        if (ingressperfEndtime > kubeburnerEndtime){
            END_TIME = ingressperfEndtime
        }
        else {
            END_TIME = kubeburnerEndtime
        }
    }
    else {
        if (params.WORKLOAD == 'ingress-perf') {
            START_TIME = ingressperfStarttime
            END_TIME = ingressperfEndtime
        }
        else {
            START_TIME = kubeburnerStarttime
            END_TIME = kubeburnerEndtime
        }
    }

    env.UUID = UUID
    env.JENKINS_BUILD = JENKINS_BUILD
    env.STARTDATEUNIXTIMESTAMP = START_TIME
    env.ENDDATEUNIXTIMESTAMP = END_TIME
    currentBuild.description += "<b>UUID:</b> ${env.UUID}<br/>"
    currentBuild.description += "<b>STARTDATEUNIXTIMESTAMP:</b> ${env.STARTDATEUNIXTIMESTAMP}<br/>"
    currentBuild.description += "<b>ENDDATEUNIXTIMESTAMP:</b> ${env.ENDDATEUNIXTIMESTAMP}<br/>"
}
