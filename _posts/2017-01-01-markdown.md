# this is a markdown note
when I want to debug program of revel framework, I found that difficult. but the good news is I got it.
you just do following

1. revel build project1 project2
2. configuring your Edit Configurations
3. Working directory: /home/cy/go/src/project1
4. Program arguments: -importPath project1 -srcPath /home/cy/go/src/ -runMode dev
5. 在Before launch,加号 run external tool 加号 在
- Program: /home/cy/bin/revel
- Parmeters: build /home/cy/go/src/project1
- Working directory: /home/cy/bin
