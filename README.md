# Preface

## Introduction <a href="#introduction" id="introduction"></a>

This book describes the implementation, administration, and use of Transarc Corporation's AFS©, the Andrew File System. This distributed system has several attributes which make it ideally suited for use in organizations trying to manage the constantly growing amount of file data needed and produced by today's operating systems and applications.

Even though most of the academic emphasis on AFS concerns its performance and semantics, much of the benefit of the system derives from its integrated approach to administration. The ability of AFS to support many more clients per server, to decrease network loads, to provide location-transparent access to data, and to automatically fail over to replica sites is reason enough to investigate the system. But the collection of tools provided to administrators which provides behind-the-scenes control of the distributed system is equally important.

Over the last decade, AFS's use has steadily increased so that by now many hundreds of sites around the world are full-fledged members of an on-line, global, distributed file system. These sites tend to be quite large because AFS is particularly optimized to support the file storage needs of thousands of users. Yet given the ever-increasing sales of computers and their voracious disk appetites, mature solutions to medium- and large-scale computing sites will be needed by more and more people.

## Audience And Scope <a href="#audience-and-scope" id="audience-and-scope"></a>

When you purchase AFS, you'll receive several manuals that come with the package. This book is not a replacement for serious reading of that official documentation. There are a multitude of options for almost all processes and utilities in the AFS command suite. Rather than catalog each argument, I hope to provide the reasons behind some of the design decisions which otherwise may appear arbitrary or counterintuitive.

How and why the AFS clients and servers work as they do is the scope of this book. The following chapters describe the newer features of the system and highlight the latest advances with some explanations of their purpose; they do not describe the precise syntax or option to every possible command. For that, you can use the manual set supplied by Transarc, the developer of the commercial AFS product.

The examples and suggestions for managing AFS are shown using the standard UNIX© command-line interface; the AFS server programs are available only for UNIX servers, so knowledge of basic UNIX commands and operations is a prerequisite for using the system.

If you've used NFS© or NetWare™ before and have managed desktops, file servers, and disks, you may be wondering what all the fuss is about; after all, these services have been around for years. But AFS is designed quite differently from these other file systems for reasons that make sense but take some explaining.

There's no simple linear way to discuss the mechanisms to manage and use AFS because most parts of the system are dependent on each other. I will try to create some order out of this interdependence at the risk of using some terms that are not fully described until later. For example, access control lists - used to detail who can access which files - are mentioned early on but are not defined until the middle of the book. The definition isn't required in the early stages, as long as you can trust that eventually the specifics will be explained.

Structurally, we'll begin with a broad overview and gradually introduce more and more detail, moving from the central servers to the distributed client desktops.

## Contents <a href="#contents" id="contents"></a>

The first two chapters describe AFS in general terms.

Chapter 1 provides a general overview of the benefits of AFS, why it was developed in the first place, its particular design and drawbacks, and information about Transarc. Administrators, managers, and users will gain an understanding of the system from Chapter 1.

Chapter 2 introduces much of the technical vocabulary of AFS and describes its client/server architecture, including the caching protocol and file server organization. Here, the architecture described in Chapter 1 is put into concrete terms of the protocol used by clients and servers.

The next four chapters are devoted to the administrative specifics required to install and run an AFS cell.

Chapter 3 discusses basic server management by itemizing the processes, operations, and systems that make the servers work, with special emphasis on the internal distributed database protocol. These issues are of concern when setting up an AFS cell.

Chapter 4 discusses AFS volume management issues, the very heart of AFS administration. While AFS is primarily thought of as a protocol by which client workstations access file data, the system's real value is in its support for large-scale administration. This chapter describes how AFS volumes are administered to create the global file namespace, to provide access to replicated data, and to provide users with on-line access to a backup version of their files.

Chapter 5 describes client configurations and includes suggestions for optimizing desktop access to AFS. The chapter also introduces the package client administration tool as well as two products that allow PCs to access AFS - Transarc's port to Windows™ NT and Platinum Technology's PC-Enterprise™.

Chapter 6 describes the user authentication process, management of the Kerberos database, and the use of standard MIT Kerberos, versions 4 and 5.

Chapter 7 details the user and developer view of AFS, including logging in to AFS, access controls, group management, and the slight differences in file system semantics. There's also a brief description of how PC users manage their AFS files with Transarc's NT port. Users not interested in administration and management can refer to this chapter for examples of how to use AFS.

Chapter 8 focuses on Transarc's built-in archiving system, ad hoc volume dump tools, and mechanisms to save other critical configuration data. It also includes information on integrating AFS with commercial backup systems.

Chapter 9 returns to the subject of overall administration, with more information on AFS server administrative responsibilities, insights into the Kerberos authentication system, providing access for NFS clients, issues regarding the installation of third-party software, and comments on a few of the public and commercial AFS administrative tools now available.

Chapter 10 continues the subject of management with a discussion of AFS's debugging and monitoring tools. The chapter focuses in depth on advanced issues of administering UNIX-based AFS servers.

Chapter 11 describes large-site administration and presents four case studies of successful use of AFS by global or otherwise interesting sites.

Chapter 12 concludes the book with some thoughts on how to evaluate and begin using AFS at your organization, and what alternative file systems are up to. Managers and administrators will gain a greater appreciation for the potential scope of an AFS implementation project.

Appendix A is a command-by-command description of the major AFS tools including all subcommands. As you read through the book, you can refer to this appendix for a reminder of the purpose of any commands. This listing also includes information on the precise authentication required to run each command.

## Typographic Conventions <a href="#typographic-conventions" id="typographic-conventions"></a>

System operations are shown as command-line input and output, such as:

```
        $ fs mkm alice  user.alice
        $ fs lsmount alice
        'alice' is a mount point for '#user.alice'
```

For people unfamiliar with UNIX, `$` is the standard UNIX Bourne shell prompt. Following UNIX convention, when you have logged in as the superuser or root, the shell prompt is `#`. These command sequences were executed on a Sun© workstation running the Solaris™ operating system with AFS version 3.4a and pasted into the text of this book. Output from AFS commands on other systems should be identical apart from dates and other ephemeral data; output from standard UNIX commands may differ more or less depending on your particular desktop operating system. Liberal use is made of standard programs such as `date`, `touch` and shell output redirection to demonstrate file creation. The `cat` program is often used to demonstrate that the files in question were created successfully.

The hypothetical organization used as an example throughout the examples, HQ, and its domain name, hq.firm, were non-existent at the time this book was written.

## Acknowledgements <a href="#acknowledgements" id="acknowledgements"></a>

In writing this book I am indebted to the software engineers who put AFS together, first (and still) at Carnegie Mellon University and later at Transarc Corporation; to the computing community which has freely shared experiences and workarounds; to the University of Michigan's Center for Information Technology Integration, where some of my best friends work and play; and to the Union Bank of Switzerland for their support. Many people in these groups helped review this book; over the years, I've learned much about AFS, but their dedicated oversight, support, and occasional drinks at corner tables in smoky bars have made this a much better book. Among them, I'd like to thank John Brautigam, Bill Doster, Ted Hanss, Peter Honeyman, Tony Mauro, Tom Menner, Anne Perella, Jim Rees, Dave Richardson, Lyle Seaman, and Mike Stolarchuk.

I'd also like to thank the staff at Prentice Hall who have guided my first effort at book-length writing from conception to publication.

And one nonacknowledgment: While I know many people from Transarc and have happily used and administered AFS over the course of many years, this book is not a product of Transarc, Corporation. Certain members of Transarc's staff, past and present, have helped me with the myriad system details, but the content and any errors in the book are mine alone. Notes and additional information on AFS can be found at my web site, `http://www.netrc.com`.
