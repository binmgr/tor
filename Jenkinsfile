// Jenkins declarative pipeline for the tor static binary builder.
// Verification build: mirrors the gates of GitHub Actions, Gitea/Forgejo, and GitLab CI.
pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timeout(time: 4, unit: 'HOURS')
    }

    environment {
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'binmgr/tor'
    }

    stages {
        stage('Get Tor version') {
            steps {
                script {
                    if (env.TOR_VERSION_OVERRIDE?.trim()) {
                        env.TOR_VERSION = env.TOR_VERSION_OVERRIDE
                    } else {
                        env.TOR_VERSION = sh(
                            returnStdout: true,
                            script: """curl -fsSL https://dist.torproject.org/ | grep -oE 'tor-[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+\\.tar\\.gz' | sed -e 's/^tor-//' -e 's/\\.tar\\.gz$//' | sort -uV | tail -1"""
                        ).trim()
                    }
                    echo "Building Tor ${env.TOR_VERSION}"
                }
            }
        }

        stage('Security') {
            parallel {
                stage('Secret scan') {
                    agent {
                        docker {
                            image 'trufflesecurity/trufflehog:latest'
                            args '--entrypoint=""'
                        }
                    }
                    steps {
                        sh 'trufflehog git file://. --since-commit HEAD~1 --fail --only-verified'
                    }
                }
            }
        }

        stage('Build binaries') {
            matrix {
                axes {
                    axis {
                        name 'TARGET'
                        values 'linux-amd64', 'linux-arm64', 'windows-amd64', 'windows-arm64', 'darwin-amd64', 'darwin-arm64', 'freebsd-amd64', 'freebsd-arm64'
                    }
                }
                stages {
                    stage('Build') {
                        agent {
                            docker {
                                image "${env.REGISTRY}/${env.IMAGE_NAME}:build"
                                reuseNode true
                            }
                        }
                        steps {
                            sh "build-tor ${TARGET} ${TOR_VERSION} out"
                            archiveArtifacts artifacts: 'out/**', allowEmptyArchive: false
                        }
                    }
                }
            }
        }

        stage('Release') {
            when { buildingTag() }
            steps {
                echo 'Release publishing is performed by the hosted provider (GitHub/GitLab/Gitea/Forgejo) — Jenkins build is verification-only.'
            }
        }
    }
}
