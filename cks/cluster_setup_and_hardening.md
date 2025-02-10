2375 port -> docker is running

4C's
1.Cloud -> misconfigurations
2.Cluster -> 
3.Container -> 
4.Code ->

##CIS(Center for Internet Security) Benchmarks:
The CIS-CAT Lite tool helps organizations check if their systems meet security benchmarks recommended by CIS. It can assess the configurations of operating systems, software, and network devices against these security standards.
The key features of CIS-CAT Lite include:
    1.Security Configuration Assessment: It assesses system configurations based on CIS benchmarks.
    2.Automated Reporting: It generates a report detailing the system's compliance status.
    3.Free Version: CIS-CAT Lite is available for free, but it provides fewer features compared to the full version, CIS-CAT Pro.

##We have installed the CIS-CAT Pro Assessor tool called Assessor-CLI, under /root.
Please run the assessment with the Assessor-CLI.sh script inside Assessor directory and generate a report called index.html in the output directory /var/www/html/.Once done, the report can be viewed using the Assessment Report tab located above the terminal.
Run the test in interactive mode and use below settings:
Benchmarks/Data-Stream Collections: : CIS Ubuntu Linux 20.04 LTS Benchmark v2.0.1
Profile : Level 1 - Server
```bash
cd /root/Assessor
sh ./Assessor-CLI.sh -i -rd /var/www/html/ -nts -rp index

##Select the below after running it
#Benchmarks/Data-Stream Collections: CIS Ubuntu Linux 20.04 LTS Benchmark v2.0.1
#Profile: Level 1 - Server
```
CIS-CAT Lite doesn't support k8s
CIS-CAT pro support k8s

###Kube-bench -> OpenSource CIS benchmark tool for k8s
Install-> container,pod in k8s cluster, install kube-bench binary
```
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.4.0/kube-bench_0.4.0_linux_amd64.tar.gz -o kube-bench_0.4.0_linux_amd64.tar.gz
tar -xvf kube-bench_0.4.0_linux_amd64.tar.gz
```

Run a kube-bench test now and see the results
Run below command to run kube bench
```
 ./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml
```
