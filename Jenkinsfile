@Library('flexy') _

// global variables for pipeline
NETOBSERV_MUST_GATHER_IMAGE = 'quay.io/netobserv/must-gather'
BASELINE_UPDATE_USERS = ['auto', 'aramesha', 'memodi']

// rename build
def userId = currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause)?.userId
if (userId) {
    currentBuild.displayName = userId
} else {
    currentBuild.displayName = 'auto'
}

pipeline {
    environment {
        def NOO_BUNDLE_VERSION = ''
        def BENCHMARK_CSV_LOG = "$WORKSPACE/e2e-benchmarking/benchmark_csv.log"
        def BENCHMARK_COMP_LOG = "$WORKSPACE/e2e-benchmarking/benchmark_comp.log"
    }

    // job runs on specified agent
    agent { label params.JENKINS_AGENT_LABEL }

    // set timeout and enable coloring in console output
    options {
        timeout(time: 6, unit: 'HOURS')
        ansiColor('xterm')
    }

    // job parameters
    parameters {
        string(
            name: 'JENKINS_AGENT_LABEL',
            defaultValue: 'oc416',
            description: 'Label of Jenkins agent to execute job'
        )
        string(
            name: 'FLEXY_BUILD_NUMBER',
            defaultValue: '',
            description: '''
                Build number of Flexy job that installed the cluster<br/>
                <b>Note that only AWS clusters are supported at the moment</b>
            '''
        )
        separator(
            name: 'CLUSTER_CONFIG_OPTIONS',
            sectionHeader: 'OpenShift Cluster Configuration Options',
            sectionHeaderStyle: '''
                font-size: 14px;
                font-weight: bold;
                font-family: 'Orienta', sans-serif;
            '''
        )
        string(
            name: 'WORKER_COUNT',
            defaultValue: '0',
            description: '''
                Total Worker count desired to scale the cluster to<br/>
                If set to '0' no scaling will occur<br/>
                You can also directly scale the cluster up (or down) yourself with <a href=https://mastern-jenkins-csb-openshift-qe.apps.ocp-c1.prod.psi.redhat.com/job/scale-ci/job/e2e-benchmarking-multibranch-pipeline/job/cluster-workers-scaling/>this job</a>
            '''
        )
        booleanParam(
            name: 'INFRA_WORKLOAD_INSTALL',
            defaultValue: false,
            description: 'Install workload and infrastructure nodes even if less than 50 nodes'
        )
        booleanParam(
            name: 'INSTALL_DITTYBOPPER',
            defaultValue: false,
            description: 'Value to install dittybopper dashboards to cluster'
        )
        string(
            name: 'DITTYBOPPER_REPO',
            defaultValue: 'https://github.com/cloud-bulldozer/performance-dashboards.git',
            description: 'You can change this to point to your fork if needed'
        )
        string(
            name: 'DITTYBOPPER_REPO_BRANCH',
            defaultValue: 'master',
            description: 'You can change this to point to a branch on your fork if needed'
        )
        separator(
            name: 'LOKI_CONFIG_OPTIONS',
            sectionHeader: 'Loki Operator Configuration Options',
            sectionHeaderStyle: '''
                font-size: 14px;
                font-weight: bold;
                font-family: 'Orienta', sans-serif;
            '''
        )
        choice(
            name: 'LOKI_OPERATOR',
            choices: ['None', 'Released', 'Unreleased'],
            description: '''
                You can use either the latest released or unreleased version of Loki Operator:<br/>
                <b>Released</b> installs the <b>latest released downstream</b> version of the operator, i.e. what is available to customers<br/>
                <b>Unreleased</b> installs the <b>latest unreleased downstream</b> version of the operator, i.e. the most recent internal bundle<br/>
                If <b>None</b> is selected the installation will be skipped and Loki will disabled in Flowcollector
            '''
        )
        choice(
            name: 'LOKISTACK_SIZE',
            choices: ['1x.demo', '1x.extra-small', '1x.small', '1x.medium'],
            description: '''
                Depending on size of cluster, use following guidance to choose LokiStack size:<br/>
                1x.demo - 3 nodes<br/>
                1x.extra-small - 10 nodes<br/>
                1x.small - 25 or 65 nodes<br/>
                1x.medium - 120 nodes<br/>
            '''
        )
        separator(
            name: 'NETOBSERV_CONFIG_OPTIONS',
            sectionHeader: 'Network Observability Configuration Options',
            sectionHeaderStyle: '''
                font-size: 14px;
                font-weight: bold;
                font-family: 'Orienta', sans-serif;
            '''
        )
        choice(
            name: 'INSTALLATION_SOURCE',
            choices: ['None', 'Official', 'Internal', 'Source'],
            description: '''
                Network Observability can be installed from the following sources:<br/>
                <b>Official</b> installs the <b>latest released downstream</b> version of the operator, i.e. what is available to customers<br/>
                <b>Internal</b> installs the <b>latest unreleased downstream</b> version of the operator, i.e. the most recent internal bundle<br/>
                <b>Source</b> installs the <b>latest unreleased upstream</b> version of the operator, i.e. directly from the main branch of the upstream source code<br/>
                If <b>None</b> is selected the Operator installation and flowcollector configuration will be skipped
            '''
        )
        string(
            name: 'IIB_OVERRIDE',
            defaultValue: '',
            description: '''
                If using Internal installation, you can specify here a specific internal index image to use in the CatalogSource rathar than using the most recent bundle<br/>
                These IDs can be found in CVP emails under 'Index Image Location' section<br/>
                e.g. <b>450360</b>
            '''
        )
        string(
            name: 'OPERATOR_PREMERGE_OVERRIDE',
            defaultValue: '',
            description: '''
                If using Source installation, you can specify here a specific premerge Operator bundle image to use in the CatalogSource rather than using the main branch<br/>
                These SHA hashes can be found in PR's after adding the label '/ok-to-test'<br/>
                e.g. <b>e2bdef6</b>
            '''
        )
        string(
            name: 'FLP_PREMERGE_OVERRIDE',
            defaultValue: '',
            description: '''
                You can specify here a specific FLP premerge image to use rather than using the operator defined image<br/>
                These SHA hashes can be found in FLP PR's after adding the label '/ok-to-test'<br/>
                e.g. <b>e2bdef6</b>
            '''
        )
        string(
            name: 'EBPF_PREMERGE_OVERRIDE',
            defaultValue: '',
            description: '''
                You can specify here a specific eBPF premerge image to use rather than using the operator defined image<br/>
                These SHA hashes can be found in eBPF PR's after adding the label '/ok-to-test'<br/>
                e.g. <b>e2bdef6</b>
            '''
        )
        string(
            name: 'PLUGIN_PREMERGE_OVERRIDE',
            defaultValue: '',
            description: '''
                You can specify here a specific ConsolePlugin premerge image rather than using the operator defined image<br/>
                These SHA hashes can be found in ConsolePlugin PR's after adding the label '/ok-to-test'<br/>
                e.g. <b>e2bdef6</b>
            '''
        )
        string(
            name: 'CONTROLLER_MEMORY_LIMIT',
            defaultValue: '',
            description: 'Note that 800Mi = 800 mebibytes, i.e. 0.8 Gi'
        )
        separator(
            name: 'FLOWCOLLECTOR_CONFIG_OPTIONS',
            sectionHeader: 'Flowcollector Configuration Options',
            sectionHeaderStyle: '''
                font-size: 14px;
                font-weight: bold;
                font-family: 'Orienta', sans-serif;
            '''
        )
        string(
            name: 'EBPF_SAMPLING_RATE',
            defaultValue: '1',
            description: 'Rate at which to sample flows'
        )
        string(
            name: 'EBPF_CACHE_MAXFLOWS',
            defaultValue: '100000',
            description: 'eBPF max flows that can be cached before evicting'
        )
        string(
            name: 'EBPF_MEMORY_LIMIT',
            defaultValue: '',
            description: 'Note that 800Mi = 800 mebibytes, i.e. 0.8 Gi'
        )
        booleanParam(
            name: 'EBPF_PRIVILEGED',
            defaultValue: false,
            description: 'Check this box to run ebpf-agent in privileged mode'
        )
        string(
            name: 'FLP_MEMORY_LIMIT',
            defaultValue: '',
            description: 'Note that 800Mi = 800 mebibytes, i.e. 0.8 Gi'
        )
        booleanParam(
            name: 'ENABLE_KAFKA',
            defaultValue: true,
            description: 'Check this box to setup Kafka for NetObserv or to update Kafka configs even if it is already installed'
        )
        choice(
            name: 'TOPIC_PARTITIONS',
            choices: [6, 10, 24, 48],
            description: '''
                Number of Kafka Topic Partitions. 48 Partitions are used for all Perf testing scenarios<br/>
            '''
        )
        string(
            name: 'FLP_KAFKA_REPLICAS',
            defaultValue: '',
            description: '''
                Replicas should be at least half the number of Kafka TOPIC_PARTITIONS and should not exceed number of TOPIC_PARTITIONS or number of nodes:<br/>
                <b>Leave this empty if you're triggering standard perf-workloads, default of 6, 12 and 18 FLP_KAFKA_REPLICAS will be used for node-density-heavy, ingress-perf and cluster-density-v2 workloads respectively</b><br/>
                3 - for non-perf testing environments<br/>
                <b>Use this field to overwrite default FLP_KAFKA_REPLICAS</b><br/>
            '''
        )
        separator(
            name: 'WORKLOAD_CONFIG_OPTIONS',
            sectionHeader: 'Workload Configuration Options',
            sectionHeaderStyle: '''
                font-size: 14px;
                font-weight: bold;
                font-family: 'Orienta', sans-serif;
            '''
        )
        choice(
            name: 'WORKLOAD',
            choices: ['None', 'cluster-density-v2', 'node-density-heavy', 'ingress-perf'],
            description: '''
                Workload to run on Netobserv-enabled cluster<br/>
                "cluster-density-v2" and "node-density-heavy" options will trigger "kube-burner-ocp" job<br/>
                "ingress-perf" will trigger "ingress-perf" job<br/>
                "None" will run no workload<br/>
                For additional guidance on configuring workloads, see <a href=https://docs.google.com/spreadsheets/d/1DdFiJkCMA4c35WQT2SWXbdiHeCCcAZYjnv6wNpsEhIA/edit?usp=sharing#gid=1506806462>here</a>
            '''
        )
        string(
            name: 'VARIABLE',
            defaultValue: '1000',
            description: '''
                This variable configures parameter needed for each type of workload. <b>Not used by <b><a href=https://github.com/cloud-bulldozer/e2e-benchmarking/blob/master/workloads/ingress-perf/README.md>ingress-perf</a> workload</b>.<br/>
                <a href=https://github.com/cloud-bulldozer/e2e-benchmarking/blob/master/workloads/kube-burner/README.md>cluster-density-v2</a>: This will export JOB_ITERATIONS env variable; set to 4 * num_workers. This variable sets the number of iterations to perform (1 namespace per iteration).<br/>
                <a href=https://github.com/cloud-bulldozer/e2e-benchmarking/blob/master/workloads/kube-burner/README.md>node-density-heavy</a>: This will export PODS_PER_NODE env variable; set to 200, work up to 250. Creates this number of applications proportional to the calculated number of pods / 2<br/>
                Read <a href=https://github.com/openshift-qe/ocp-qe-perfscale-ci/tree/kube-burner/README.md>here</a> for details about each variable
            '''
        )
        string(
            name: 'NODE_COUNT',
            defaultValue: '3',
            description: '''
                Only for <b>node-density-heavy</b><br/>
                Should be the number of worker nodes on your cluster (after scaling)
            '''
        )
        booleanParam(
            name: 'CERBERUS_CHECK',
            defaultValue: false,
            description: 'Check cluster health status after workload runs'
        )
        string(
            name: 'E2E_BENCHMARKING_REPO',
            defaultValue: 'https://github.com/cloud-bulldozer/e2e-benchmarking',
            description: 'You can change this to point to your fork if needed'
        )
        string(
            name: 'E2E_BENCHMARKING_REPO_BRANCH',
            defaultValue: 'master',
            description: 'You can change this to point to a branch on your fork if needed.'
        )
        separator(
            name: 'NOPE_TOUCHSTONE_CONFIG_OPTIONS',
            sectionHeader: 'NOPE and Touchstone Configuration Options',
            sectionHeaderStyle: '''
                font-size: 14px;
                font-weight: bold;
                font-family: 'Orienta', sans-serif;
            '''
        )
        booleanParam(
            name: 'NOPE',
            defaultValue: false,
            description: '''
                Check this box to run the NOPE tool after the workload completes<br/>
                If the tool successfully uploads results to Elasticsearch, Touchstone will be run afterword
            '''
        )
        booleanParam(
            name: 'GEN_CSV',
            defaultValue: false,
            description: '''
                Boolean to create Google Sheets with statistical and comparison data where applicable<br/>
                Note this will only run if the NOPE tool successfully uploads to Elasticsearch
            '''
        )
        booleanParam(
            name: 'NOPE_DUMP_ONLY',
            defaultValue: false,
            description: '''
                Check this box to dump data collected by the NOPE tool to a file without uploading to Elasticsearch<br/>
                Touchstone will <b>not</b> be run if this box is checked
            '''
        )
        booleanParam(
            name: 'NOPE_DEBUG',
            defaultValue: false,
            description: 'Check this box to run the NOPE tool in debug mode for additional logging'
        )
        string(
            name: 'NOPE_JIRA',
            defaultValue: '',
            description: '''
                Add a NETOBSERV Jira ticket or any context to be associated with this Perf run, <b>must set when INSTALLATION_SOURCE = None</b><br/>
            '''
        )
        booleanParam(
            name: 'RUN_BASELINE_COMPARISON',
            defaultValue: false,
            description: '''
                If the workload completes successfully and is uploaded to Elasticsearch, checking this box will configure Touchstone to run a comparison between the job run and a baseline UUID<br/>
                By default, the NOPE tool will be used to fetch the most recent baseline for the given workload from Elasticsearch<br/>
                Alternatively, you can specify a specific baseline UUID to run the comparison with using the text box below
            '''
        )
        string(
            name: 'BASELINE_UUID_OVERRIDE',
            defaultValue: '',
            description: '''
                Override baseline UUID to compare this run to<br/>
                Note you will still need to check the box above to run the comparison<br/>
                Using this override will casue the job to not automatically update the Elasticsearch baseline even is the comparsion is a success
            '''
        )
        separator(
            name: 'CLEANUP_OPTIONS',
            sectionHeader: 'Cleanup Options',
            sectionHeaderStyle: '''
                font-size: 14px;
                font-weight: bold;
                font-family: 'Orienta', sans-serif;
            '''
        )
        booleanParam(
            name: 'NUKEOBSERV',
            defaultValue: false,
            description: '''
                Check this box to completely remove NetObserv and all related resources at the end of the pipeline run<br/>
                This includes the AWS S3 bucket, Kafka (if applicable), LokiStack, and flowcollector
            '''
        )
        booleanParam(
            name: 'DESTROY_CLUSTER',
            defaultValue: false,
            description: '''
                Check this box to destroy the cluster at the end of the pipeline run<br/>
                Ensure external resources such as AWS S3 buckets are destroyed as well
            '''
        )
    }

    stages {
        stage('Capture NetObserv Operator Bundle version'){
            when {
                expression { params.INSTALLATION_SOURCE == "Official" || params.INSTALLATION_SOURCE == "Internal" }
            }
            agent { 
                // run on upshift_docker agent because
                // opm needs registry login
                label "upshift_docker"
            }
            steps {
                script {
                    // copy artifacts from Flexy install
                    copyArtifacts(
                        fingerprintArtifacts: true,
                        projectName: 'ocp-common/Flexy-install',
                        selector: specific(params.FLEXY_BUILD_NUMBER),
                        target: 'flexy-artifacts'
                    )
                    withCredentials(
                        [usernamePassword(credentialsId: 'c1802784-0f74-4b35-99fb-32dfa9a207ad', usernameVariable: 'QUAY_USER', passwordVariable: 'QUAY_PASSWORD')]){
                        sh 'podman login -u $QUAY_USER -p $QUAY_PASSWORD quay.io'
                    }
                    withCredentials([usernamePassword(credentialsId: 'db799772-7708-4ed0-bfe7-0fddfc8088eb', usernameVariable: 'REG_REDHAT_USER', passwordVariable: 'REG_REDHAT_PASSWORD')]){
                        sh 'podman login -u $REG_REDHAT_USER -p ${REG_REDHAT_PASSWORD} registry.redhat.io'
                    }
                    withCredentials([usernamePassword(credentialsId: 'brew-registry-osbs-mirror', usernameVariable: 'BREW_USER', passwordVariable: 'BREW_PASSWORD')]){ 
                        sh 'podman login -u $BREW_USER -p $BREW_PASSWORD brew.registry.redhat.io'
                    }
                    withCredentials([usernamePassword(credentialsId: '41c2dd39-aad7-4f07-afec-efc052b450f5', usernameVariable: 'REG_STAGE_USER', passwordVariable: 'REG_STAGE_PASSWORD')]){
                        sh 'podman login -u $REG_STAGE_USER -p $REG_STAGE_PASSWORD registry.stage.redhat.io'
                    }

                    NOO_BUNDLE_VERSION=sh(returnStdout: true, script: '''
                            #!/usr/bin/env bash
                            mkdir -p ~/.kube
                            cp ${WORKSPACE}/flexy-artifacts/workdir/install-dir/auth/kubeconfig ~/.kube/config
                            RELEASE=$(oc get pods -l app=netobserv-operator -o jsonpath='{.items[*].spec.containers[1].env[0].value}' -A | cut -d 'v' -f 3)

                            # source from 4.12 because jenkins agent are on RHEL 8
                            curl -sL "https://mirror2.openshift.com/pub/openshift-v4/x86_64/clients/ocp/latest-4.12/opm-linux.tar.gz" -o opm-linux.tar.gz
                            tar xf opm-linux.tar.gz
                            ./opm alpha list bundles quay.io/openshift-qe-optional-operators/aosqe-index:v4.17 netobserv-operator | grep 1.7.0 | awk '{print $5}'
                            BUNDLE_IMAGE=$(./opm alpha list bundles quay.io/openshift-qe-optional-operators/aosqe-index:v4.17 netobserv-operator | grep 1.7.0 | awk '{print $5}')
                            oc image info $BUNDLE_IMAGE -o json --filter-by-os linux/amd64 | jq '.config.config.Labels.url' | awk -F '/' '{print $NF}' | tr '\"' ' '
                        ''').trim()
                    if (NOO_BUNDLE_VERSION != '') {
                        println("Found NOO Bundle version: ${NOO_BUNDLE_VERSION}")
                        currentBuild.description += "NetObserv Bundle Version: <b>${NOO_BUNDLE_VERSION}</b><br/>"
                    }
                    else {
                        println("Failed to find NOO_BUNDLE_VERSION :(, comparison may not have context and may use UUID")
                    }
                }
            }
        }
    }

    post {
        always {
            println('Post Section - Always')
            archiveArtifacts(
                artifacts: 'e2e-benchmarking/utils/*.json,e2e-benchmarking/*.log',
                allowEmptyArchive: true,
                fingerprint: true
            )
            dir("/tmp"){
                archiveArtifacts(
                    artifacts: 'flowcollector.yaml',
                    allowEmptyArchive: true,
                    fingerprint: true)
            }
            dir("$workspace/ocp-qe-perfscale-ci/"){
                archiveArtifacts(
                    artifacts: 'data/**, scripts/netobserv/netobserv-dittybopper.yaml',
                    allowEmptyArchive: true,
                    fingerprint: true)
            }
        }
        failure {
            println('Post Section - Failure')
        }
        cleanup {
            script {
                // uninstall NetObserv operator and related resources if desired
                if (params.NUKEOBSERV == true) {
                    println("Deleting Network Observability operator...")
                    nukeReturnCode = sh(returnStatus: true, script: """
                        source $WORKSPACE/ocp-qe-perfscale-ci/scripts/netobserv.sh
                        nukeobserv
                    """)
                    if (nukeReturnCode.toInteger() != 0) {
                        error('Operator deletion unsuccessful - please delete manually :(')
                    }
                    else {
                        println('Operator deleted successfully :)')
                    }
                }
                if (params.DESTROY_CLUSTER == true) {
                    println("Destroying Flexy cluster...")
                    build job: 'ocp-common/Flexy-destroy', parameters: [
                        string(name: 'BUILD_NUMBER', value: params.FLEXY_BUILD_NUMBER)
                    ]
                }
            }
        }
    }
}
