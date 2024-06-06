# Installing R and overlays with packages necessary for R scripts to run on the HPC

1. Log in to the HPC. STAY in your home folder where you have writable rights!! It won't let you do this installation anywhere else due to R library writing permission!

2. Log in to the "software node". You need to enter your password again.
```
ssh software 
```

## Creating base R version 3.6

3. Create a “r.def” file
```
nano r.def
```
4. Write in the block of commands that will install base version of R and some necessary libraries and development tools
```
BootStrap: yum
OSVersion: 7
MirrorURL: http://yum-repos.hpccluster/centos/7/os/$basearch/
Include: yum


%post
yum -y install epel-release
yum -y install R
yum -y install which make gcc gcc-c++ libcurl-devel libxml2-devel openssl-devel texlive-*
```
5. Build the base R image
```
sudo singularity build r.img r.def
```
## Create an overlay image file which will contain the R packages needed for analysis.
- Default allowed overlay img size is 64Mb! Need to increase the allowed size if the packages take some space otherwise you will end up with an error "No space left on device"
```
singularity overlay create --size 4096 r-tximport.img
```
6. Now connect it to the base R image file which is version 3.6.0
```
sudo singularity shell --overlay r-tximport.img r.img
```
7. Singularity terminal opens. Write what you want to install. Include the lines before and after.
- Also, you might need to know the repository where your package is located and specify it (you can google that). But for the ones installed below, the single repository set should work (checked 5th June 2024).
```
/bin/R --vanilla <<EOF

r = getOption("repos")
r["CRAN"] = "http://cloud.r-project.org"
options(repos = r)
rm(r)

install.packages(c("ggplot2", "BiocManager","dplyr","usethis","miniUI"))
BiocManager::install(c("IRanges", "Rhdf5lib", "rhdf5", "tximportData", "tximport", "GenomeInfoDb", "DMRcaller"))

q()
EOF
```
8. After this, commands will run and install packages (it will take a while). When its finished type:
```
exit
```
### If you have non-zero exit status for some packages due to missing dependencies, you need to install them as well. Just add it to the list of packages, before the desired one that needs it.

9. Afterwards, you may log out of the software node and move both R image files wherever you need.
```
logout
```

10. Finally, you can run your image overlay over the base R image with your Rscript:
```
singularity exec --overlay /jic/scratch/groups/Philippa-Borrill/scripts/R-3.6-all.img /jic/scratch/groups/Philippa-Borrill/scripts/r-upd.img Rscript  YOUR_R_SCRIPT.R
```
singularity exec --overlay: runs the overlay R image
Rscript: basic command to run a Rscript in Linux environment 

### If you are running a longer job, this needs to be part of a proper shell script for job submission

Example:

```
#!/bin/bash
#SBATCH -p jic-medium
#SBATCH -t 0-10:00
#SBATCH -c 1
#SBATCH --mem=30000
#SBATCH -J tximport
#SBATCH --mail-type=none
#SBATCH -o tximport.out
#SBATCH -e tximport.err


singularity exec --overlay /jic/scratch/groups/Philippa-Borrill/scripts/R-3.6-all.img /jic/scratch/groups/Philippa-Borrill/scripts/r-upd.img Rscript  YOUR_R_SCRIPT.R
```

