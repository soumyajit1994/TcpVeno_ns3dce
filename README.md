#  Validate the ns-3 implementation of TCP Veno using ns-3 DCE
--------------------

Brief about the project
------------------------
TCP Veno is  targeted to improve the performance of TCP in wireless
environments. It is a combination of Vegas and Reno, and hence, named Veno. In this
project, the aim is to validate ns-3 TCP Veno implementation by comparing the results
obtained from it to those obtained by simulating Linux TCP Veno.

Table of Contents
---------------------
1. An overview
2. Steps to reproduce

1.An overview
----------------
ns-3 is a free open source project aiming to build a discrete-event
network simulator targeted for simulation research and education.
In brief, ns-3 provides models of how packet data networks work and perform, 
and provides a simulation engine for users to conduct simulation experiments. 
Some of the reasons to use ns-3 include to perform studies that are more difficult 
or not possible to perform with real systems.
[For more information click on this Link](http://www.nsnam.org)

Direct Code Execution (DCE) is a module for ns-3 that provides facilities to execute,
within ns-3, existing implementations of userspace and kernelspace network protocols or 
applications without source code changes.

The main goal of this project is to have a Linux like TCP implementation in ns-3.
In this project we have used ns-3 docker container.

               
 2.Steps to reproduce
 -------------------------------------------------------------
1. We have installed ns-3 dce using docker image as we can create as many containers as we want (which is nothing but different copies of ns-3 DCE) and we can experiment different techniques in different containers simultaneously on one system.Ubuntu 16.04 or higher version is recommended.

2. After installing docker [installing docker](https://docs.docker.com/engine/install/ubuntu/), pull the docker image to install NS-3 DCE.

                sudo docker pull thehajime/ns-3-dce

3. Run docker to create a container.This will take us to a particular container where the ns-3 dce can be found.

             sudo docker run -i -t thehajime/ns-3-dce /bin/bash

4. Install vim in docker container 

             sudo apt install vim

5. Copy & replace the files  `tcp-veno.cc`,`tcp-congestion-ops.cc`,`tcp-congestion-ops.h` inside `source/ns-3-dev/src/internet/model/ ` .

6. Keep the topologies and scripts from folder Topology in home directory in local machine.

7. Whenever any changes is made in DEV, you need to build the dce using the below set of commands, otherwise the changes will not be reflected in your setup
       
       7a)Go to “ns3dce@4c788588614e:~/dce-linux-dev$” and set environment variables :
        --------------------------------------------------------------------------------------------------------------
             - export BAKE_HOME=`pwd`/bake
             - export PATH=$PATH:$BAKE_HOME
             - export PYTHONPATH=$PYTHONPATH:$BAKE_HOME
       
       7b) Configure DCE :
       -------------------------
               - bake.py configure -e dce-linux-dev
       
       7c) Configure DCE :
       -------------------------   
               - bake.py build -vvv

8. Go to `source/ns-3-dce/`  . Inside the container and run following command to check weather install correctly or not(by running iperf)` ./waf –run dce-iperf`.If it is built successfully DCE is correctly installed.
9. Copy dumbbell topology(dumbbelltopologyns3receiver.cc) inside `ns-3-dce/example/`   using

             sudo docker cp dumbbelltopologyns3receiver.cc your docker name:/home/ns3dce/dce-linux-dev/source/ns-3-dce/example

10. Update the wscript in ns-3-dce using 

              sudo docker cp wscript your docker name:/home/ns3dce/dce-linux-dev/source/ns-3-dce

11. Run the above 2 commands on a different terminal outside the container.

12. Run the dumbbell topology in linux stack of ns-3-dce in the directory `source/ns-3-dce/`   using
 
         ./waf --run ”dumbbelltopologyns3receiver --stack=linux --queue_disc_type=FifoQueueDisc --WindowScaling=true -- Sack=true --stopTime=300 --delAckCount=1 --BQL=true --linux_prot=veno” 

11. Copy the `parse_cwnd.py` file in `ns-3-dce/utils/ ` using the  command in other terminal

        sudo docker cp parse_cwnd.py your docker name:/home/ns3dce/dce-linux-dev/source/ns-3-dce/utils 

       and run there using   `python parse_cwnd.py 2 2`.

12. Running this file will collect the traces generated by running the dumbbell topology example on linux stack. This will create a   folder `cwnd_data` inside `ns-3-dce/results/dumbbell-topology`  where there will be  `A-linux.plotme` file.
 
13. Make one folder `overlapped` inside `ns-3-dce/results/dumbell-topology/` and copy the  A-linux.plotme file from `cwnd_data` to `overlapped` folder.
 
14. Run the dumbbell topology in ns3 stack using the command

          ./waf --run "dumbbelltopologyns3receiver --stack=ns3 -queue_disc_type=FifoQueueDisc --WindowScaling=true 
          --Sack=true --stopTime=300 --delAckCount=1 --BQL=true --transport_prot=TcpVeno --recovery=TcpPrrRecovery"
          
           
     This will create a new timestamp folder under `ns-3-dce/results/dumbbell-topology/` and under  the latest timestamp folder, find a folder `cwndTraces` where `A- ns3.plotme` file will be generated. Copy this file in `overlapped` folder.
          
15. Now copy the `overlap-gnuplotscriptCwnd` script inside `overlapped` using 

           sudo docker  cp overlap-gnuplotscriptCwnd your docker name:/home/ns3dce/dce-linux-dev/source/ns-3-dce /results/dumbbell-topology/overlapped 
        
       and install `gnuplot` in directory `overlapped` using 

           sudo apt-get install gnuplot
           
       Then run
 
            gnuplot overlap-gnuplotscriptCwnd
 
16. Above steps will create a `CwndA.png` file inside overlapped which we can copy to home directory using 

             sudo docker cp your docker name:/home/ns3dce/dce-linux-dev/source/ns-3-dce/results/dumbbell-topology/overlapped/CwndA.png .
             
 17. Docker names can be found using `sudo docker ps -a`
         
               
               
