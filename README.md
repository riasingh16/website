# website
This repository contains a website like quora
It will contain the questions and answers 

 
name: Push

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Creating 3 Node GKE Cluster
      run: |
        chmod 755 ./build/github/stages/1-gke-setup/cluster-setup
        ./build/github/stages/1-gke-setup/cluster-setup
      env:
        SDK_TOKEN: ${{ secrets.SDK_TOKEN }}
        PROJECT_NAME: ${{ secrets.PROJECT_NAME }}

    - name: Creating nginx deployment application
      run: kubectl apply -f litmus/nginx.yml

    - name: Set config
      run: echo ::set-env name=KUBE_CONFIG_DATA::$(base64 -w 0 ~/.kube/config)
           
    - name: Running Litmus pod delete chaos experiment
      uses: mayadata-io/github-chaos-actions@v0.1.0
      env:
        ##If litmus is not installed
        INSTALL_LITMUS: true
        ##Give application info under chaos
        APP_NS: default
        APP_LABEL: run=nginx
        APP_KIND: deployment
        EXPERIMENT_NAME: pod-delete
        ##Custom image can also been used
        EXPERIMENT_IMAGE: litmuschaos/ansible-runner:latest        
        TOTAL_CHAOS_DURATION: 30
        CHAOS_INTERVAL: 10
        FORCE: false
        ##Select true if you want to uninstall litmus after chaos
        LITMUS_CLEANUP: true


    - name: Deleting GKE Cluster
      run: |
        chmod 755 ./build/github/stages/2-gke-cleanup/cluster-cleanup
        ./build/github/stages/2-gke-cleanup/cluster-cleanup
      env:
        SDK_TOKEN: ${{ secrets.SDK_TOKEN }}
        PROJECT_NAME: ${{ secrets.PROJECT_NAME }}  
