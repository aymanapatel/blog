---
title: "Land of Shell Scripting"
seoTitle: "What is the difference between bash, sh, dash, zsh. And powershell is"
seoDescription: "What is the difference between bash, sh, dash, zsh. And powershell is powerful!"
datePublished: Sun Mar 31 2024 11:23:17 GMT+0000 (Coordinated Universal Time)
cuid: cluffngms000008jv431i4f1j
slug: land-of-shell-scripting
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711884316671/5abd7c46-3d41-4884-aee0-43e060cc5a7d.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1711884326689/6342b925-44c4-44e2-b834-b7babbbac4b3.png
tags: devops, shell, shell-scripting

---

Shell scripts are one of those things that seem easy and portable on the surface, but then you start writing and realize there is much to think about while writing the scripts. You need to see what Shell environment (zsh, bash, dash, fish) you are using, what OS you are on (Windows, Mac, Fedora, Debian, Alpine) etc. Shell script that works in 1 environment is not guaranteed to execute in a different environment.

# Shell Scripting vs shell environment

As a developer on a UNIX-like OS, there are 2 things we typically interact with in the terminal.

1. Shell Prompt
    
2. Shell Scripts
    

**Shell Prompt** is a terminal interface that can be customised based on the shell prompt used. These are a matter of preference and taste. Tools like ZSH allow you to add some convenient features out-of-the-box such as syntax highlighting, themes, metadata to show things like git branch, time, RAM usage etc and much more. BASH also has these features but these are harder and more time-consuming to setup as compared to ZSH and the Oh-my-zsh plugin ecosystem. There are alternatives such as [Startship](https://starship.rs/) that provides a faster ZSH+oh-my-zsh startup by leveraging rust.

**Shell Scripts** are scripting languages that are used to run some tasks which could include server management, git hooks, packages installation and much more. This blog will dive deep into this topic as there are a lot of variations and nuances in writing and executing shell scripts across Operating systems such as Linux, MacOS and Windows.

# History of Shells

1. **Thompson Shell**: Developed by Ken Thompson in Bell Labs. Was meant for shell environment only with no scripting language.
    
2. **C Shell**: Tenex C Shell (`csh`) was created by Bill Joy in 1978. It'sain contribution was to add a scripting language, which was missing from the original Thompson shell.
    
3. **Korn Shell**: Create by David Korn in 1983 and was inspired by CSH's programming capability using a scripting language. One advantage of KSH was that it was backwards compatible with the traditional original Thompson shell. It was open-source in 2000.
    

KSH is still found in many OS environment. CSH is not recommended as it is not compatible with the Bourne Shell.

# POSIX compliance for shells

> Why we need to know about POSIX?

Every UNIX-like OS implements the POSIX standard. Hence, on the surface level the POSIX compliant APIs will exist but due to different implementation-level details for every flavour of the OS, the command might differ in reality. For example, the Regex implementation in Alpine (most popular Docker image due to its tiny size), differs to that of a traditional distribution. Why is it the case? Alpine does not implement the traditional GNU Utils. Instead, it relies on Busybox which is a stripped down reimplementation of the Unix utilities. The reason why Busybox is so small is that it removes a lot of legacy code of the traditunal GNU Utils package.

## What it takes for shell to be POSIX compliant

> Trivia: POSIX (1992) came after Bourne Shell (1976) and even BASH(1990)

Bourne Shell (discussed below) is strictly POSIX compliant. For a shell to be POSIX-compliant it is defined in [here](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)

In UNIX, all interactions are based on standard IO (`STDIO`) data streams that can be pipelined. What this means is that all communications between Shell and Kernel follow simple data byte streams which can be fed from 1 program to another.

![[Bashism_Arch.png]]( align="left")

`STDIO` has 3 parts (in shell's architecture)

1. `STDIN`: Standard input for taking input as text
    
2. `STDOUT`: Standard output which stores the output in stdout stream
    
3. `STDERR`: Standard error
    

These 3 are the building blocks of shell interactions with a **pipeline** architecture where output of one shell can be fed into of another shell script.

# Current Shells in existence

The Shells discussed earlier were stepping stones to the shells we have today. The shells below are the ones that are widely used.

If you want to check what is the shell being used, run this command

```shell
ls -l /usr/bin/sh
```

## Bourne Shell aka sh

Created by Stephen Bourne at Bell Labs in 1979. It was distributed as part of UNIX Version 7 release.

The OG shell that is still used. Some are of the opinion that `sh` is the 1 true shell and all others are not recommended (except for dash). Bourne shell was open sourced much later, due to which dash(which closely resembled Bourne Shell) was made the default in Debian-based distributions.

Bourne shell is one of the early mainstream shells that provided programming constricts such as loops, flow control (if else), mathematical operations, variables and bi-directional communication between shell and the commands that is inside the shell program.

## bash (Bourne-again shell)

> bash is not sh, it is **Born Again SHell**

Written by Brain Fox as part of GNU as a free-software alternative to the Bourne shell. It is a superset of bourne shell which might not be strictly POSIX compliant.

You can see the not-so-accurate venn diagram on how the different shells are in terms of compatibility

![](../!images/Shells.png align="left")

### bashisms

Bashisms are the extensions to Shell's POSIX compliance to make it a bit easier to write scripts. In other words, it is a set of convenient syntax which is not possible in traditional Bourne Shells

For example, you cannot create arrays in traditional Bourne shell, but in bash you can create it like this

```sh
my_arr = (1 2 4 "hi" 5)
```

Yes, you can change types in the same array!

There is a great [wiki](https://mywiki.wooledge.org/Bashism) that goes into detail on the bashism quirks and how they can be implemented in a more strictly-POSIX compliant shell like Bourne Shell or Dash. (Sadly, arrays are tricky to do in Bourne Shell and dash!)

Here we have a sample script to check bashisms

```bash
#!/bin/bash

## 1. Conditional Bracket bashism
x=1
if [[ $x -eq 1 ]]
then
  echo "bashism worked for [[" 
else
  echo "bashism failed for [[" 
fi

## 2. Equality Bashism
y="string_cmp"
if [ $y == "string_cmp" ]
then
  echo "bashism worked for string == "
else
 echo "bashism failed for string == "
fi


## 2. Array bashism
arr1=(1 2 "hi" 4)
for str in "${arr1[@]}"; do
  echo "$str"
done
if [ $? = 0 ]                                                                               then                                                                                          echo "Shell failed. See above for error"                                                  fi
```

![](../images/Bashism_zsh.png align="left")

## zsh

> zsh can also be used as a scripting language and not just the traditional Shell prompt environment

Due to strong copy left licensing with GPLv3 by the Free Software Foundation, many operating systems like Mac and Kali have been moving away from bash as their default shell, and have started using zsh as the default shell environment.

## dash (Debian Almquist shell)

Usually found in Debian distributions. DASH is the most pure form of Bourne Shell. The syntax is the exact same as that of Bourne Shell. As it does not have a lot of features that bash has, it is faster to execute than bash.

The screenshot below shows a Debian distro whose default shell symlinks to dash.

![](../!images/Debian-Shell-DASH.png align="left")

You can see the errors of the bashism script we used earlier

![](../images/Bashism_DASH.png align="left")

Error output:

```shell
debian@debian:~/Desktop/dash$ ./bash_script.sh 
./bash_script.sh: 11: [[: not found
bashism failed for [[
./bash_script.sh: 20: [: string_cmp: unexpected operator
bashism failed for string == 
./bash_script.sh: 29: Syntax error: "(" unexpected
```

# Powershell

Even though Powershell might cause some people to irk from their past experiences with Command Prompt and Windows Development in general, latest Powershell has pretty strong capabilities.

All UNIX type shell have a **pipeline** architecture where output of one command can be an input of another command. Powershell differs in this architecture by embracing .NET's Object-Oriented roots. It works with objects instead of pipes which provides new use cases in shell scripting.

We will look into 2 examples of how powershell aids in using objects as a uilding block of writing shell scripts

## Powershell CSV example

1. Import CSV Data and see the Object Type definition
    

```shell
PS powershell-example> $csv_data = Import-Csv ./employee_data.csv                                                                                                                                    
PS powershell-example> $csv_data | Get-Member                                                                                                                                                        

   TypeName: System.Management.Automation.PSCustomObject

Name        MemberType   Definition
----        ----------   ----------
Equals      Method       bool Equals(System.Object obj)
GetHashCode Method       int GetHashCode()
GetType     Method       type GetType()
ToString    Method       string ToString()
Age         NoteProperty string Age=33
EID         NoteProperty string EID=e102
Title       NoteProperty string Title=manager
User        NoteProperty string User=john
```

2. Sorting the CSV by User
    

```bash

PS powershell-example> $csv_data | Sort-Object -Property User -Descending                                                                                                                            

User   Age Title    EID
----   --- -----    ---
neha   31  engineer e032
john   33  manager  e102
javier 36  director e222
```

3. Casting the type of Age from string to number!
    

```bash
PS powershell-example> $casted_csv_data = $csv_data | Select-Object User, Title, EID, @{NAME="Age";Expression={[int]$_.Age}}                                                                         
PS powershell-example> $casted_csv_data | Get-Member                                                                                                                                                 

   TypeName: Selected.System.Management.Automation.PSCustomObject

Name        MemberType   Definition
----        ----------   ----------
Equals      Method       bool Equals(System.Object obj)
GetHashCode Method       int GetHashCode()
GetType     Method       type GetType()
ToString    Method       string ToString()
Age         NoteProperty System.Int32 Age=33
EID         NoteProperty string EID=e102
Title       NoteProperty string Title=manager
User        NoteProperty string User=john
```

![](../!images/Powershell_CSV.png align="left")

As you can see, it is quite a tool for writing shell scripts. UNIX would have worked till step 2, but due to its STDIO architecture, it cannot do more than just texts whereas powershell do strongly typed queries.

## Powershell JSON example

Here we are importing a JSON file and changing a surname of an employee. As .NET objects can be used, it can be done with simple human readable commands.

![](../!images/Powershell_JSON.png align="left")

```bash
PS powershell-example> $emp_json = Get-Content ./employee_data.json | ConvertFrom-Json
PS powershell-example> $emp_json.accounts.users.neha.surname = "Patil"             PS powershell-example> $emp_json | ConvertTo-Json -Depth 4 | Out-File ./employee_data_converted.json 
PS powershell-example> cat ./employee_data_converted.json
{
  "accounts": {
    "users": [
      {
        "john": {
          "givenName": "John",
          "surname": "Doe",
          "department": "Finance"
        },
        "javier": {
          "givenName": "Javier",
          "surname": "Hernandez",
          "department": "Marketing"
        },
        "neha": {
          "givenName": "neha",
          "surname": "Patil",
          "department": "Product"
        }
      }
    ]
  }
}
```

Other file formats that is part of Powershell utilities include

1. Markdown - Reference [doc here](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertfrom-markdown?view=powershell-7.4)
    
2. HTML - Reference [doc here](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertto-html?view=powershell-7.4)
    
3. XML - Reference [doc here](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertto-xml?view=powershell-7.4) Hence, Powershell can act as a powerful tool in writing scripts and can be used in some circumstances to parse CSV, JSON, XML, HTML etc.
    

There are more things to cover such as ZSH, exhaustive list of bashisms, Shell indifferences due to different GNU Utils libraries etc.

> She SHells C Shells at the ZSH ore