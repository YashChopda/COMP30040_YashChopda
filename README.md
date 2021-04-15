# Final Year Project COMP30040_YashChopda
Author - Yash Chopda, Third Year student studying Artificial Intelligence with Industrial Experience in the University of Manchester. 
Project Supervisior - Dr. Lucas Cordeiro


-------------------
#### Aim: To investigate low level implementation errors related to digital controllers and hardware compatibility. 
       To check how digital controllers are susceptible to violations according to fixed point implementations.
       To reproduce and validate the results produced by DSVerifier with external tools like CBMC v5.11 with Mini-
       Sat Solver, ESBMC v6.6.0 with Boolector and Verifuzz using CBMC v5.10 with Glucose Syrup solver.
     
-------------------
#### Benchmarks:
The results in the experiment done are based on a real quadcopter attitude control system used for for aerial surveillance.
The properties verified are Limit Cycle, Overflow by Saturation, Overflow by Wrap-Around and Stability. Verification is used for
5, 10 and 15 bounds.

--------------------
#### Pre-requisites- System Requirements:
1) >= gcc 5.4
2) Eigen Library to be installed depending on distribution.
3) Install time limit using:
     - sudo apt install timelimit 

Note: for all verifications, the time limit used is 3600s

---------------------
#### Repository Structure:
benchmarks- It contains tr2018 with Controller_Implementations containing the ANSI C implementations. Results_DSVERIFIER contains 
results for properties LimitCycle, Overflow-Saturate-Mode, Overflow-WrapAround-Mode and Stability in which you can find result logs for
k=5,10 and 15 with the tables containing verification results. The folders LimitCycle, Overflow-Saturate-Mode, Overflow-WrapAround-Mode and Stability in tr2018 contains Verification bounds and each contains 3 folders that are external tools used. Eact external tool folder contains 3 realizations, DFI, DFII and TDFII which contains the preprocessed c files for those tools and the result logs contains all the logs(results) for the verification.

---------------------
#### Tools:
##### DSVERIFIER
It is a verification tool for digital systems. In this project we are using dsverifier-v2.0.3-esbmc-v4.0-cbmc-5.6.

To download the latest verison and to set up the tool go to-
http://www.dsverifier.org

For verification using DSVERIFIER it is necessary for a file to be in the following ANSI-C format:

   	```c
   	#include <dsverifier.h>

	digital_system ds = {
		.b = { 1.5, -0.5 },
		.b_size = 2,
		.a = { 1.0, 0.0 },
		.a_size = 2,
		.sample_time = 0.02
	};
	
	implementation impl = {
		.int_bits = 2,
		.frac_bits = 14,
		.max = 1.0,
		.min = -1.0
	};
	```
	
	Execution - ./dsverifier <file> --property <prp> --realisation <r> --x-size <k> 
		     --bmc <modelChecker> --solver <s> --timeout <t>
		     
		     where <file> is the controller implementation file to be verified, <prp> is the property to be verified,
		     <r> is the realisation form, <k>	is the verification bound, <s> is the solver used, <t> is time in seconds for
		     program to timeout and <modelChecker> is the bounded model checker used like CBMC (v5.6) or ESBMC (v4.0.0).
		     
		     Properties: 
		     LIMITCYCLE, STABILITY, OVERFLOW.
		     While verifying overflow using Saturation and WrapAround, we use --overflow-mode <o> where o can be SATURATE and
		      WRAPAROUND. 
	             
	             Realisation forms: DFI, DFII, TDFII
	             
	             Solver used is boolector, other available are z3 and yices.
	             
	             To generate counterexample data add --show-ce-data.

	
##### CBMC v5.11 using Mini-SAT solver - external tool 
Download : https://www.cprover.org/cbmc/download/cbmc-5-11-linux-64.tgz
	   extract the tool and open the directory and check ./cbmc --version and if it says 5.11 (cbmc-5.11) then it is correctly set up.
	
	
	Perprocessing:
	
	Firstly, for every verification, we need create a C file of controller implementations which is compatible for cbmc.
	We need to preprocess it using following command:
	
	gcc -E <file> -DBMC=CBMC -I <bmc path> -DREALIZATION=<r> - DPROPERTY=<prp> -DX_SIZE =<k> > <output file>	
        
        where <file> is the original controller implementation file to be verified, <prp> is the property to be verified,
	<r> is the realisation form, <k>is the verification bound, <bmc path> is the path of bmc folder of dsverifierv2.0.3 and 
	<output file> is the resulting preprocessed c file which will be used to verify using CBMC v5.11.
	     
	Properties: 
	LIMITCYCLE, STABILITY, OVERFLOW.
	While verifying overflow using Saturation and WrapAround, we use --overflow-mode <o> where o can be SATURATE and
	WRAPAROUND. 
	             
	Realisation forms: DFI, DFII, TDFII
	             	
	example: gcc -E benchmarks/tr2018/Controller_Implemetations/ds-01-impl1.c -DBMC=CBMC -I /home/COMP30040_YashChopda/bmc 
	-DREALIZATION=DFI -DPROPERTY=LIMIT_CYCLE -DX_SIZE =10 > myFile.c 
	
	Execution: 
	
	timelimit <t> ./cbmc <file> --trace > <result.out>
	
	where <t> is time in seconds for program to timeout, <file> is the preprocessed file obtained for that particular verification
	and <result.out> is output file with result obtained. trace is used to get counter example.
	
	example: timelimit -t3600 ./cbmc ~/COMP30040_YashChopda/benchmarks/tr2018/LimitCycle/K=5/CBMC/DFI/ds-01-impl1.c --trace > 
	         ~/COMP30040_YashChopda/benchmarks/tr2018/LimitCycle/K=5/CBMC/DFII/result_logs/ds-01-impl1.out
		
	To make execution of benchmarks(of this project only) easier one can use following command:
	In the directory:
		  timelimit -t3600 ./cbmc ~/COMP30040_YashChopda/benchmarks/tr2018/<prp>/K=<k>/CBMC/<r>/<implementation>.c --trace > 
	         ~/COMP30040_YashChopda/benchmarks/tr2018/<prp>/K=<k>/CBMC/<r>/result_logs/<implementation>.out
	where <prp> is LimitCycle, Overflow-Saturate-Mode, Overflow-WrapAround-Mode and Stability, <k> is 5, 10, 15,
	<r> is DFI, DFII, TDFII and <implementation> is controller implementation name example ds-01-impl1.
	
##### ESBMC v5.11 using Boolector solver - external tool 

	Installation:
	   Run ./ESBMC-Linux.sh , if it doesnt execute because of error : permission denied do chmod u+x ESBMC-Linux.sh 
	   After accepting everything you will get ESBMCv6.6.0.
	   
	Perprocessing:
	
	Firstly, for every verification, we need create a C file of controller implementations which is compatible for esbmc.
	We need to preprocess it using following command:
	
	gcc -E <file> -DBMC=ESBMC -I <bmc path> -DREALIZATION=<r> - DPROPERTY=<prp> -DX_SIZE =<k> > <output file>	
        
        where <file> is the original controller implementation file to be verified, <prp> is the property to be verified,
	<r> is the realisation form, <k>is the verification bound, <bmc path> is the path of bmc folder of dsverifierv2.0.3 and 
	<output file> is the resulting preprocessed c file which will be used to verify using CBMC v5.11.
	     
	Properties: 
	LIMITCYCLE, STABILITY, OVERFLOW.
	While verifying overflow using Saturation and WrapAround, we use --overflow-mode <o> where o can be SATURATE and
	WRAPAROUND. 
	             
	Realisation forms: DFI, DFII, TDFII
	             	
	example: gcc -E benchmarks/tr2018/Controller_Implemetations/ds-01-impl1.c -DBMC=ESBMC -I /home/COMP30040_YashChopda/bmc 
	-DREALIZATION=DFI -DPROPERTY=LIMIT_CYCLE -DX_SIZE =10 > myFile.c 
	
	Execution: 
	
	 Enter following command in bin file.
	./esbmc <file> --show-cex-data --timeout <t> > <result.out>
	
	where <t> is time in seconds for program to timeout, <file> is the preprocessed file obtained for that particular verification
	and <result.out> is output file with result obtained. trace is used to get counter example.
	
	example: ./esbmc ~/COMP30040_YashChopda/benchmarks/tr2018/Stability/K=15/ESBMC/TDFII/ds-01-impl1.c --show-cex --timeout 3600 > 
	~/COMP30040_YashChopda/benchmarks/tr2018/Stability/K=15/ESBMC/TDFII/result_logs/ds-01-impl1.out 
		
	To make execution of benchmarks(of this project only) easier one can use following command:
	In the directory:
		  ./esbmc ~/COMP30040_YashChopda/benchmarks/tr2018/<prp>/K=<k>/ESBMC/<r>/<implementation>.c --show-cex --timeout 3600 > 
	         ~/COMP30040_YashChopda/benchmarks/tr2018/<prp>/K=<k>/ESBMC/<r>/result_logs/<implementation>.out
	where <prp> is LimitCycle, Overflow-Saturate-Mode, Overflow-WrapAround-Mode and Stability, <k> is 5, 10, 15,
	<r> is DFI, DFII, TDFII and <implementation> is controller implementation name example ds-01-impl1.

##### Verifuzz using CBMC v5.10 Glucose Syrup solver - external tool 
Download : https://gitlab.com/sosy-lab/sv-comp/archives-2020/-/raw/master/2020/verifuzz.zip
	   extract the tool and open the directory and check ./bin/cbmc --version and if it says 5.10 (cbmc-5.10) then cbmc is correctly 
	   set up.
	
	
	Perprocessing:
	
	Firstly, for every verification, we need create a C file of controller implementations which is compatible for cbmc.
	We need to preprocess it using following command:
	
	gcc -E <file> -DBMC=CBMC -I <bmc path> -DREALIZATION=<r> - DPROPERTY=<prp> -DX_SIZE =<k> > <output file>	
        
        where <file> is the original controller implementation file to be verified, <prp> is the property to be verified,
	<r> is the realisation form, <k>is the verification bound, <bmc path> is the path of bmc folder of dsverifierv2.0.3 and 
	<output file> is the resulting preprocessed c file which will be used to verify using CBMC v5.10.
	     
	Properties: 
	LIMITCYCLE, STABILITY, OVERFLOW.
	While verifying overflow using Saturation and WrapAround, we use --overflow-mode <o> where o can be SATURATE and
	WRAPAROUND. 
	             
	Realisation forms: DFI, DFII, TDFII
	             	
	example: gcc -E benchmarks/tr2018/Controller_Implemetations/ds-01-impl1.c -DBMC=CBMC -I /home/COMP30040_YashChopda/bmc 
	-DREALIZATION=DFI -DPROPERTY=LIMIT_CYCLE -DX_SIZE =10 > myFile.c 
	
	Execution: 
	
	timelimit <t> ./bin/cbmc <file> --trace > <result.out>
	
	where <t> is time in seconds for program to timeout, <file> is the preprocessed file obtained for that particular verification
	and <result.out> is output file with result obtained. trace is used to get counter example.
	
	example: timelimit -t3600 ./bin/cbmc ~/COMP30040_YashChopda/benchmarks/tr2018/LimitCycle/K=5/Verifuzz/DFI/ds-01-impl1.c --trace > 
	         ~/COMP30040_YashChopda/benchmarks/tr2018/LimitCycle/K=5/Verifuzz/DFII/result_logs/ds-01-impl1.out
		
	To make execution of benchmarks(of this project only) easier one can use following command:
	In the directory:
		  timelimit -t3600 ./bin/cbmc ~/COMP30040_YashChopda/benchmarks/tr2018/<prp>/K=<k>/Verifuzz/<r>/<implementation>.c --
		  trace > ~/COMP30040_YashChopda/benchmarks/tr2018/<prp>/K=<k>/Verifuzz/<r>/result_logs/<implementation>.out
	where <prp> is LimitCycle, Overflow-Saturate-Mode, Overflow-WrapAround-Mode and Stability, <k> is 5, 10, 15,
	<r> is DFI, DFII, TDFII and <implementation> is controller implementation name example ds-01-impl1.
	
	
	
	
	
