# Architectural Overview

Let's allow the devil's advocate to begin: People and applications have been using distributed file systems for several years. The total cost of hardware and especially disk drives has been plummeting for the last decade and looks like it will continue to do so. Computers are sharing data through local-area and wide-area networks now more than ever. Files, sounds, still and moving pictures, and interactive forms and games are being delivered through the World Wide Web, in some cases directly to consumer's television sets. So why do we need yet another file system?

There are two immediate responses: One: files may not be the sexiest of media, but they are where the overwhelming majority of information we come into contact with is stored. And two: there are still plenty of problems with the current implementations. Do any of the following scenarios apply to your systems?

* A desktop is attached to a remote server that stores production applications, but the network is down so the user can't run anything.
* A desktop user is having a problem with an application. After searching though the local disk, the user discovers some outdated system binaries.
* A new project team has been hired and they'll need their own server because the current server and network are already overloaded.
* An important new application is available, but you can't run it because you can't find it on your current server.
* Your project has run out of shared disk space. Administrators will have to wait until evening to shut down the machine and add a disk. They're grumbling because they've stayed late for several consecutive nights just to add more disk drives to other servers.
* You want to share a file with a new employee but uncertainties about Internet use prevent both of you from sharing the easy way. Finally, you give up, copy the file to a floppy, and walk over to the new employee's computer.
* You want to share a file with the entire organization. You send an e-mail to the administrators asking them to copy the file to a general directory. You're not sure when this file will ever become generally available.
* A user accidentally deleted their mail folder. It will take a several hours of an administrator's time to retrieve this file from backup tape.
* A new employee joins the company. After finally getting an ID and home directory, she finds that this information has not been propagated to some of the systems she'd like to use.
* A user is moving to an office on another floor for a few days. How will he find his home directory on a different desktop? How much network traffic will this connection generate?
* A user is moving permanently to a new floor or office. The administrators schedule another late shift so they can copy over her files and update the location maps.

And consider if the above problems are happening in the office across the street. Or across the country.

Most users are happy enough with distributed system technologies if they can just get connected and perform some function. The richness of the AFS solution is not just that it enables the easy sharing of information, but that it also builds a management infrastructure that supports ever-increasing sharing without a similar increase in costs. It's not just a way to share a file; it's a way for an organization to control its distributed file services.

The benefits of AFS include:

* High scalability because of protocol efficiencies
* Ease of sharing via its single namespace
* Reliable access to replicated data
* Centralized administration model based on distributed services
* Improved security model for access and group management
* Superior support for dataless clients and desktops of different architectures

These features are not a random collection of tricks stuffed into a single package. They are the result of a conscious effort to build a sustainable distributed file system that can support medium- to large-scale enterprises. Whereas other file system products have evolved to fill a market, AFS was engineered to solve a set of problems - the kinds of problems, as you can see, that affect most organizations today.

## Beginnings <a href="#beginnings" id="beginnings"></a>

As with many UNIX products, the AFS developers were university researchers who were charged with providing solutions to their users. In this case, the campus was Carnegie Mellon University in the early 1980s, the researchers worked for the central computing center, and the problem was how to easily share file data between people and departments.

CMU was looking to alleviate the difficulties departments encountered as individuals dealt with disk drives, file locations, duplicate data, and service outages. The group wanted to provide central storage for file resources such that the entire campus, some thousands of workstations and users, could have access to the same files. This wish raised many issues: How many servers would be needed? Could the network traffic be supported with the existing LAN and WAN infrastructure of the day? With so many files, how would users navigate the file namespace? How many administrators would be needed to manage the system? What about failures?

Few technologies were available to help with these issues and most were proprietary to a particular vendor's operating system. Sun's Network File System stood apart from the rest in that it was bundled on many different platforms and the implementation was available for study.

But CMU soon found inefficiencies with the NFS architecture: For typical systems of the time, only a few dozen clients could be supported; there was no systematic security system; because clients could mount practically anything, anywhere they wanted, the same files would wind up with different names on different clients, or worse, no two clients ever saw the exact same set of files. And with the small memory-only cache, certain benchmarks that performed many reads and writes of file data to and from the servers failed to complete at all at certain loads.

More importantly, the CMU researchers recognized that the costs attributed to enterprise file management include not only the hardware and software that store and deliver the data but also the administration costs to oversee and manage these processes. In short, the strengths of NFS - its small scale and flexibility - proved to be its weakness. This was not the architecture to solve the problems CMU had identified.

Luckily, the University was in the habit of undertaking serious computing research projects with an eye toward using those same systems in production at their campus - the Andrew Message Service for multimedia mail and the Mach operating system are two of the more notable examples. So, the center decided to engineer their own system, one that could handle either a small or large site without the hefty increases in the number of systems and support staff otherwise necessary. This new system became known as the Andrew File System, or AFS. (The eponymous name recognizes both Andrew Carnegie and Andrew Mellon, the primary benefactors of CMU).

To create AFS, the researchers made several key assumptions, foremost being the decision to exploit the increasingly powerful processing capacity of the desktop computers themselves. In the modern era of computing, the disparity between server and client power is diminishing; why not take advantage of often underused clients to help take the load off the disk servers and networks? The designers rigorously examined any way to reduce network protocol packets and server processing. In all cases, the semantics of the file system operations were to be kept as close as possible to local behavior, but certain optimizations might be adopted if the changes made sense in a distributed system.

Because CMU is an academic institution, its security needs may appear to be less than those of other companies; however, with the constantly changing user population and the relatively open campus environment, the problems of inappropriate access to data are perhaps greater than in many other organizations. A new security model would have to be implemented; one that used the public WAN but relied on cryptographically secure communications to authenticate the identities of users. The inherently untrustworthy desktops would never confirm a user's identity; only the trusted AFS security servers could do that.

Another goal was to build complete location transparency into the system: operators should be able to manage the storage devices without having to concern themselves with manually updating desktop configurations. Any changes to server configurations or the ultimate physical location of a set of files should be automatically noticed by interested desktops without manual intervention by administrators or users. And as administrators and users manipulated the file system, clients must have guaranteed access to the latest data.

A key factor in the design of AFS was the result of the latest file system research. It turned out that even in a distributed organization involved in intensive software development, file access patterns were not random. Overwhelmingly, files were opened and read through from beginning to end. And though software development means writing files and creating new executables, files are still read three times more often than they are written. Further, usage of the file system shows high locality of reference: files that were read recently are far more likely to be read again. And finally, these files were most often quite small, only tens to hundreds of kilobytes.

These results of file system studies suggested that aggressive caching of files by clients would prove successful in reducing server load. The designers of AFS decided to fully embrace client-side caching with quite a large disk cache and a guaranteed consistency model. While this made the AFS servers and clients more complicated to engineer, the hypothesis was that the servers would be able to support a large campus environment without a significant investment in specialized hardware or networks.

## Benefits Of AFS <a href="#benefits-of-afs" id="benefits-of-afs"></a>

The first version of AFS was completed in 1984. The results were impressive: versus NFS, the savings in protocol efficiencies permitted from five to ten times as many clients per server and reduced network traffic some two-thirds. And, even better, this capability also translated into faster client response times.

The earliest versions comprised user-made processes named Venus and Vice, names which live on in a few places in AFS. In the version 2 release of AFS in 1986, these pieces were placed inside the operating system itself for even better performance.

After a few revisions, AFS became the primary file storage system at CMU. Significantly, because it was designed to meet the real-world needs of a large, distributed organization, the system implements many policies directly rather than presenting a toolkit for administrators to build their own system. These policies allow fewer operations staff to support larger number of clients and users; again, this was a specific design goal, and the designers weighed a variety of design choices to achieve these high levels of scalability.

Figure 1-1 is a high-level view of the components that make AFS work. At the top is a typical desktop client computer that caches all information from the system onto its local disk; on the right are the various servers that respond to requests to fetch and store data.

\[\[Figure 1-1: The AFS Architecture]]

The most interesting part of this architecture is the central AFS file namespace. While all the file and directory names in the namespace are related to physical data on the servers, the names are visible in a single, abstract, enterprisewide, and consistent directory structure. The servers perform all their work in concert to provide a seamless file namespace visible by all desktops and all users. Any changes to the namespace - any new files or changed data - are publicized to clients immediately without per-client administration.

We've seen how client caching can reduce network traffic and server load. The hard part of caching is determining when cached data is invalid. In AFS, a server manages all invalidation processing by tracking which clients are using which files. When one AFS client writes a new copy of a file to a server, the server promises to call back all other workstations that have cached that file, with a brief notice to disregard that piece of cached data on subsequent requests.

Caching reduces client/server traffic enormously. When, for example, a client runs a Perl script, the Perl executable program must be read and used. In AFS, this use automatically causes the program to be cached. Now, for every new Perl script run ever after, the client is able to use the cached version. Because it uses its local copy, not a single packet is put on the network and the server is not interrupted to satisfy requests for commonly used data.

When a new version of the Perl executable is inevitably installed, the server storing the program will call back all clients it knows have cached the program and give them an explicit indication that the file has been updated. On the next execution of this program, the client will realize that the local copy is out of date and quickly fetch the newer version from the server.

Compare this mechanism with a caching Web brower: browsers also store recently accessed files in a cache, but the browser has no idea if or when that data becomes out of date. The browser must either contact the server on each access and check if a new version of the page has been installed - a check which adds additional network traffic and interrupts the server - or it must blindly assume that the page is not going to be updated for some period of time, in which case, the user may see out-of-date data until the client sees fit to check the server again. The AFS model incorporates proactive notification of new versions to all interested clients. Chapter 2 goes into more detail on how this protocol works and scales to large numbers of clients.

The fact that servers must keep track of which clients are reading what files is indeed a burden. And in AFS, a few optimizations are in place to reduce the complexity of tracking all that data and recovering from failures. But the one-time cost in developing a complex consistency model is more than offset by the day-in-day-out savings in server processing time and network load. These consistency guarantees are not constructed simply to provide the same view of the file system to all clients but to support large caches with a minimum load. AFS clients can cache their data in either kernel memory or on local disk, the latter often leading to cache sizes of several hundred megabytes. This scheme represents several hundred megabytes of data that will not be thrashing across the network or be redundantly accessed.

Next, the NFS architecture of allowing clients to mount server file systems practically anywhere in their local namespace permits great flexibility, but in large enterprises, the problem is not that users need or want to create their own directory structures but that users can't find data that is supposed to be shared globally. This problem is solved by integrating file location discovery with the file access protocol.

In Figure 1-1, you can see that a client contacts the cached, abstract namespace first. Directions to the actual storage locations of the files and directories are stored in the namespace itself; only after a client discovers these hints will that client contact a server directly to download the desired data. AFS maintains a database mapping named collections of files and their locations; as clients enter new subtrees supported by new collections, they contact the database servers, discover the new locations, and then contact the appropriate file server to fetch the correct data.

Conventionally, applications enter the AFS namespace by changing to the directory named `/afs`. As applications navigate through that well-known directory name, clients are regularly discovering location hints and determining the actual names of the servers and disk locations of the files. That location information is also cached, just as file data is, to reduce future requests to a minimum.

The huge benefit for you as an administrator (and therefore, eventually, users) is that as the locations of files are manipulated because of system housekeeping needs, the file and directory namespace remains the same. Those clients that have accessed a specific part of the file tree are called back with new location information, and any further need for file or directory information is satisfied from the new servers and disks. You don't need to run any lengthy or extraneous commands to push out this information to clients; clients inherently have access to the entire distributed file system by asking where data is and then fetching the data as one integrated operation.

Because all clients navigate the same file namespace, all changes to the files or directories are visible to all clients all the time. No extra administration need be done on a client to access different parts of the `/afs` file tree: everything under that directory is consistently visible to all clients at all times.

Also, besides being stored on physical disks, files are stored in logical containers called volumes. These collections of file subtrees, normally invisible to users, are the unit of administrative control in AFS. Typically, a package of related files is stored in a single volume - a user's home directory, the common binaries for an architecture, a version of a production application. An administrator can apportion a chunk of physical disk to a new volume and then assign the volume to represent a subtree of the AFS namespace, much as a disk partition controls a piece of a UNIX or PC file system.

These connections between path names and volumes allow administrators to enlarge any area of the namespace at will without undue concern about the location of each set of files. And through the technology out of which volumes are built, you can move a volume's files from server to server without the user ever seeing an interruption of service.

Moreover, you can make snapshots of a volume's data available in the file system; users themselves can trivially retrieve yesterday's files rather than requesting a tape retrieval of every accidental deletion.

Best of all, you can replicate volumes. Typically, replication is done for system or other production binaries such as the Perl executable or other widely used programs of data which are infrequently updated. This replication, like most AFS administration, can be performed in the background, behind the scenes, invisible to the user population, and yet used to provide a highly available system. Here, "highly available" implies a higher level of reliability and failure recovery than is otherwise available.

When clients need access to replicated file data, all locations that have been administered to contain copies of the file are returned to the client machine. In Figure 1-1, you can see that the files underneath `/afs/hq.firm/solaris/bin` are apparently stored on each of three different disks. As the client accesses file in that directory, if there is any sort of outage, such as a network problem or server crash, the client simply goes to the next server on its list. This on-line fail over has proven of critical importance to institutions that have installed AFS. And the easy availability of reliable access to important data reduces the need for more additional expensive fault-tolerant servers and redundant networking.

Note that the fail over mechanism is how clients manage to find access to critical data. Equally important is the capability added by the CMU engineers for administrators to replicate the data in the first place. Again, to compare this with NFS, nothing in the NFS protocol manages failures. Even when you've managed to repoint a client at a new NFS server following some disaster, you have got no guarantee that what you're pointing at is an exact copy of the data - if you're lucky, someone might have managed to manually copy the right files at the right time between systems. The AFS system gives you an integrated, centrally controlled capability to manage replication of file data.

For the most part, you perform all these procedures on-line, transparently to users. For example, AFS administrators routinely move a user's home directory from one system or disk to another during the middle of the day with virtually no interruption of service. You can decommission entire file servers by moving their file data to another machine, with no one the wiser. System administration does not have to be performed during off-hours, neither in the middle of the night nor on weekends.

AFS supports these procedures by centralizing the administrative databases but this does not mandate another level of bureaucracy that will encourage inefficiences. Note from the figure that the first-level administrative interface is through a simple AFS desktop client. Because the centralized mechanisms are available via a client/server command suite, with the most common tasks available as simple one-line commands, AFS allows administrators greater flexibility and power in the management of the system.

As an example, say a single-disk drive has crashed with the only copy of a user's home directory on it. AFS permits you to quickly restore that directory subtree to another disk in the file system; you can then easily connect the new location of that subtree to the old directory name, and all clients will automatically - and invisibly to users - begin to contact that new location whenever that subtree is accessed.

Given all of this centrally controlled information of file locations, server systems, and namespace management, it stands to reason that the internal databases used by AFS are of primary importance. To reduce the risks of depending on this data, the databases used by AFS are also replicated and clients can automatically fail over between the available database servers as needed. These databases are not based on any commercial relational database system; they are custom-built for AFS clients and servers and use their own caching, replication, and read/write protocol. The collection of these database and file servers and their clients is called an AFS cell.

Yet another problem faced by CMU was that of secure user authentication. The engineers looked to the Kerberos security system, designed at the Massachusetts Institute of Technology around the same time and for many of the same reasons. In a campus computing environment, an essentially unknown number of desktops are found in every nook and cranny; it is therefore unsafe to trust any of the desktops' claims about a user's identity - desktop operating systems can be too easily subverted. By storing user's secret keys in a secured database, an encrypted protocol can prove users' identities without passing those secrets around the network in the clear. Once identified, all file system transactions use this authenticated credential to check access control. The AFS system builds on this model by efficiently replicating the Kerberos servers to increase their reliability.

One part of traditional UNIX systems that has never proven adequate in large enterprises is group access to files. The UNIX permission bits allow a single group to be given a combination read/write/execute control over a specific file. The main problems with this permission system are that artificial names must be created to enable that single group to have the correct membership. And group membership has normally been controlled only by the superuser. Both features greatly reduce the effectiveness of UNIX groups.

AFS provides three solutions to managing file accesses: each directory can have a list of access controls entries with each entry containing either a user identity or a group name. So, multiple groups with different permissions can easily be accommodated. Also, the permissions allowed are extended from the usual read and write to include insertion, deletion, listing of files in the directory, file locking, and administration of the permissions themselves. Lastly, individuals can create and maintain their own groups (according to a strict naming scheme), so that group membership administration can be delegated away from the operations staff. This ability drastically reduces the administration complexity and increases the security of the file system by encouraging accurate access controls based on natural organizational groupings of users.

Seeing as how the set of AFS servers includes replicated Kerberos database servers, it is natural for standard AFS practices to mandate that the servers be kept in physically secure locations. That security consideration could be a drawback to administration procedures, but the AFS administration command suite is written to enable operators to manage all server operations from their desktops without resort to remote login. This client/server approach is natural, given the distributed nature of the system, but makes even more sense when most administration functions are operations performed on the single, global namespace. Because all changes to the namespace are visible to all clients instantaneously, administrators perform many of their day-to-day tasks by manipulating the namespace rather than by executing commands on server systems using some backdoor into the file data.

As an example of how the use of AFS depends on manipulation of the namespace, let's examine how to provide distributed file system support for clients of different architectures. In NFS, if you wanted a single, well-known file name to be used for access to the Perl executable, say, `/usr/local/bin/perl`, you could load the Solaris binaries on one file server under the name `/export/solaris/local/bin/perl` and the AIX™ binaries at `/export/aix/local/bin/perl`. Each Sun and IBM client would then have to be configured so that it mounted the correct server path name at its local path of `/usr/local/bin/`.

In AFS, the client operating system includes a hook in the internal name lookup function such that any use of the path name element `@sys` is automatically translated - on a per-client basis - into a string of characters dependent on the client's architecture. An IBM AIX version 4.1 desktop would translate `/afs/@sys/bin` into `/afs/aix_41/bin`; a Sun desktop would translate the same string as `/afs/sun4m_53/bin`. The Perl executable could therefore be loaded into the architecture-specific directory and the local path `/usr/local/bin` could be set up as a symbolic link into the AFS path `/afs/@sys/bin`. Any access to `/usr/local/bin/perl` would then be directed to the appropriate executable no matter which desktop performed the lookup.

The point of this example is that support for multiple client architectures has been transformed from an administrator's laborious management task into the simple creation of a single directory with a special name, `@sys`. While the Perl executable may be managed by administrators, any other set of files which are machine-dependent can be managed by individual developers or users. AFS decentralizes what was previously an administrator's chore into a simple path name cliche that can be implemented by anyone.

The combination of all of these benefits - scalability due to caching, single namespace, reliable data access, reduced administration overhead, security, and support for heterogeneous, dataless clients - ultimately allows us to move much of our local data from desktop clients onto AFS servers. With a high degree of confidence, we can rely on the AFS central management model with its highly available data access characteristics to support practically cheap desktop clients - clients that need almost no backing up because all interesting data is stored in the AFS system. Disks on desktops can be used to store mostly ephemeral data such as temporary files, swap space for many large processes, print and mail spool areas, and, of course, a file cache.

These desktops are called dataless because they have a disk but no interesting data is stored on them; instead, only transient data or copies of distributed data are kept local. Compare this configuration with either a diskless machine, which has no hard drive attached, or a diskfull system, which has a large disk on which are stored critical applications and other data that may or may not be available to others. Diskless machines produce a large amount of network traffic and server load because they have small caches; diskfull systems require resource-intensive per-client administration.

To effectively support dataless clients, you need a protocol which reduces network traffic as much as possible, and just as important, the protocol must have built-in fail over so that access to production data is available all the time. Even today, the only protocol which integrates these characteristics is AFS.

Another way to consider the utility of AFS is to take a very high level view of a generic computing system. In the broadest view, a traditional computer consists of a processing unit, storage, and I/O. Storage is normally of two types, transient or persistent, with the former usually implemented by volatile electronic memory and the latter by disks. In both cases, a hierarchy of caching is performed to increase performance and functionality.

For memory, a typical PC has small, fast memory on the CPU chip itself, a large off-chip cache, a much larger local memory system, and finally, a very large area of disk for temporary data. The objects stored in memory are usually the data associated with active processes, and all processes on a machine exist in a single namespace (the names are usually just the process identification number). The important point for us is that the entire complex memory hierarchy is managed automatically by the operating system. Neither users nor the processes themselves are involved in location administration. The result is seamless integration of different levels of storage with overall improved performance and low cost.

Objects stored on disk are predominantly files in a hierarchical namespace. The difference here is that the namespace and the storage devices are mostly managed manually. When no network is available, users themselves must move data from machine to machine. And often, even when a distributed file system is available, it may be that shared files and programs are stored there but the files are copied locally by innumerable custom-built scripts. This is cache management by direct manipulation.

AFS provides a way to automatically manage file storage by making local disks merely a cache for a true, enterprisewide, persistent, shared, distributed data space. Almost all aspects of the local cache are managed mechanically, using policies optimized for the general cases but that do not degrade in the large scale. Unlike the case of processes in memory, the benefit of a persistent file space is that the global set of files can be managed independently of the local desktops, but with the knowledge that any changes will automatically be visible on all desktops.

Let's recap our initial list of problems and some of the solutions provided by AFS.

* Desktops can reliably use applications that are stored on replicated servers.
* Desktops can use system files and binaries efficiently even when the files are stored remotely because, overall, the caching protocol gives near-local disk performance.
* New users do not require additional server systems because the servers and networks are doing much less work.
* All users can get to the same set of files through the single available namespace. Replication permits this data to be located close to the users who need it most without hindering use of the data by other users.
* There's far less concern about running out of disk space; as long as one disk is available on a server anywhere in the enterprise, an administrator can logically expand the namespace at any point to include that data space. Alternatively, sections of the namespace can be transparently moved around the system to disks with extra space available while users are on-line.
* Sharing files is simplified because all files in AFS are shared equally by everyone. With an integrated Kerberos authentication system and discrete access controls throughout, this sharing can be restricted or opened up as needed.
* Snapshots of yesterday's file contents are efficiently saved and are easily made available, eliminating the majority of file restoration requests.
* User identities and group information are efficiently propagated throughout the enterprise by means of a consistent distributed database model, just as file data itself is available to every desktop whenever it has changed.
* A user's move to a new floor normally requires no changes to any configurations. Because of the caching protocol used, the smallest amount of data possible will be transmitted between the servers and desktop.
* If users are moving permanently, it is easy to move their home directory and any other data with which they are closely associated to servers nearer their new desktop. Such a data move is done purely as a networking optimization; no path names are changed, and all desktops continue to see the same file namespace.
* Caring for remote systems is easy because the distributed administration model permits operators to manage and monitor servers throughout the enterprise from their own desktops.

So, what's the downside? Though AFS solves many administrative and user issues, it raises other, different problems. Only the market can tell you whether the cure is better than sickness; while this system has been overshadowed in some respects by other commercial products, the last decade has seen significant growth in the use of AFS.

## Global Filesystems <a href="#global-filesystems" id="global-filesystems"></a>

After AFS's success at CMU in the mid-80s, other universities grew interested in using the system. This collaboration was encouraged through the use of AFS itself as a means for distant sites to share information. The network protocol on which AFS is built is based on a robust, hand-crafted remote procedure call system capable of efficient transfer of file data while surviving potentially lossy WAN links. This protocol enables many organizations to access their remote cells as well as other AFS sites in a truly Internet-based and globally scalable distributed file system. Today, a typical AFS client well connected to the Internet has several dozen institutions and organizations with which it can share files - not through special-purpose file copy programs, hardcoded addresses, or browsers - but simply by changing directories and opening files, all with Kerberos-authenticated access controls.

Table 1-1 shows a small selection of cells with which a well-connected client can securely share files. This listing is effectively the visible top-level directory structure as seen on an AFS client. These entries are independent domains of AFS use, not individual machines, and each represents a handful to tens of servers, from 100 to 10,000 users, and thus, in total, literally terabytes of file data.

| AFS Cell Name           | Organization                                         |
| ----------------------- | ---------------------------------------------------- |
| transarc.com            | # Transarc Corporation                               |
| palo\_alto.hpl.hp.com   | # HP Palo Alto                                       |
| sleeper.nsa.hp.com      | # HP Cupertino                                       |
| stars.reston.unisys.com | # Paramax (Unisys), Reston, Va.                      |
| vfl.paramax.com         | # Paramax (Unisys), Paoli Research Center            |
| ibm.uk                  | # IBM UK, AIX Systems Support Centre                 |
| afs.hursley.ibm.com     | # IBM Hursley, UK                                    |
| zurich.ibm.ch           | # IBM, Zurich                                        |
| ctp.se.ibm.com          | # IBM, Chalmers, Sweden                              |
| stars.com               | # STARS Technology Center, Ballston, Va.             |
| telos.com               | # Telos Systems Group, Chantilly, Va.                |
| isl.ntt.jp              | # NTT Information and Communication Systems Labs.    |
| gr.osf.org              | # OSF Research Institute, Grenoble                   |
| ri.osf.org              | # OSF Research Institute                             |
| cern.ch                 | # European Laboratory for Particle Physics, Geneva   |
| ethz.ch                 | # Federal Institute of Technology, Zurich            |
| urz.uni-heidelberg.de   | # Universitaet Heidelberg                            |
| uni-hohenheim.de        | # University of Hohenheim                            |
| desy.de                 | # Deutsches Elektronen-Synchrotron                   |
| lrz-muenchen.de         | # Leibniz-Rechenzentrum Muenchen Germany             |
| mpa-garching.mpg.de     | # Max-Planck-Institut fuer Astrophysik               |
| ipp-garching.mpg.de     | # Max-Planck-Institut Institut fuer Plasmaphysik     |
| uni-freiburg.de         | # Albert-Ludwigs-Universitat Freiburg                |
| rhrk.uni-kl.de          | # Rechenzentrum University of Kaiserslautern         |
| rrz.uni-koeln.de        | # University of Cologne, Computing Center            |
| urz.uni-magdeburg.de    | # Otto-von-Guericke-Universitaet, Magdeburg          |
| rus.uni-stuttgart.de    | # Rechenzentrum University of Stuttgart              |
| caspur.it               | # CASPUR Inter-University Computing Consortium, Rome |
| le.caspur.it            | # Universita' di Lecce, Italia                       |
| infn.it                 | # Istituto Nazionale di Fisica Nucleare, Italia      |
| cc.keio.ac.jp           | # Keio University, Science and Technology Center     |
| sfc.keio.ac.jp          | # Keio University, Japan                             |
| postech.ac.kr           | # Pohang Universtiy of Science                       |
| nada.kth.se             | # Royal Institute of Technology, Sweden              |
| rl.ac.uk                | # Rutherford Appleton Lab, England                   |
| pegasus.cranfield.ac.uk | # Cranfield University                               |
| cs.arizona.edu          | # University of Arizona, Computer Science Dept.      |
| bu.edu                  | # Boston University                                  |
| gg.caltech.edu          | # Caltech Computer Graphics Group                    |
| cmu.edu                 | # Carnegie Mellon University                         |
| andrew.cmu.edu          | # Carnegie Mellon University, Campus                 |
| ce.cmu.edu              | # Carnegie Mellon University, Civil Eng. Dept.       |
| theory.cornell.edu      | # Cornell University Theory Center                   |
| graphics.cornell.edu    | # Cornell University Program of Computer Graphics    |
| msc.cornell.edu         | # Cornell University Materials Science Center        |
| northstar.dartmouth.edu | # Dartmouth College, Project Northstar               |
| afs1.scri.fsu.edu       | # Florida State Univeristy, Supercomputer Institute  |
| iastate.edu             | # Iowa State University                              |
| athena.mit.edu          | # Massachusetts Institute of Technology, Athena      |
| media-lab.mit.edu       | # MIT, Media Lab cell                                |
| net.mit.edu             | # MIT, Network Group cell                            |
| nd.edu                  | # University of Notre Dame                           |
| pitt.edu                | # University of Pittsburgh                           |
| psu.edu                 | # Penn State                                         |
| rpi.edu                 | # Rensselaer Polytechnic Institute                   |
| dsg.stanford.edu        | # Stanford University, Distributed Systems Group     |
| ir.stanford.edu         | # Stanford University                                |
| ece.ucdavis.edu         | # University of California, Davis campus             |
| spc.uchicago.edu        | # University of Chicago, Social Sciences             |
| ncsa.uiuc.edu           | # University of Illinois                             |
| wam.umd.edu             | # University of Maryland Network WAM Project         |
| glue.umd.edu            | # University of Maryland, Project Glue               |
| umich.edu               | # University of Michigan, Campus                     |
| citi.umich.edu          | # University of Michigan, IFS Development            |
| lsa.umich.edu           | # University of Michigan, LSA College                |
| math.lsa.umich.edu      | # University of Michigan, LSA College, Math Cell     |
| cs.unc.edu              | # University of North Carolina at Chapel Hill        |
| utah.edu                | # University of Utah Information Tech. Service       |
| cs.washington.edu       | # University of Washington Comp Sci Department       |
| wisc.edu                | # Univ. of Wisconsin-Madison, Campus                 |
| anl.gov                 | # Argonne National Laboratory                        |
| bnl.gov                 | # Brookhaven National Laboratory                     |
| fnal.gov                | # Fermi National Acclerator Laboratory               |
| ssc.gov                 | # Superconducting Supercollider Lab                  |
| hep.net                 | # US High Energy Physics Information cell            |
| cmf.nrl.navy.mil        | # Naval Research Laboratory                          |
| nrlfs1.nrl.navy.mil     | # Naval Research Laboratory                          |
| nersc.gov               | # National Energy Research Supercomputer Center      |
| alw.nih.gov             | # National Institutes of Health                      |
| nrel.gov                | # National Renewable Energy Laboratory               |
| pppl.gov                | # Princeton Plasma Physics Laboratory                |
| psc.edu                 | # Pittsburgh Supercomputing Center                   |

&#x20;Table 1-1: A Selection of Publicly Accessible AFS Cells

Although this list seems to suggest that the majority of sites are academic institutions, there are actually somewhat more commercial sites using AFS than schools. The reason that few companies are listed is simply that zealous security concerns cause most for-profit organizations to place their AFS site behind a network firewall. As long as you're using Kerberos authentication and access controls correctly, a cell is completely secure; but even so, an administrative mistake can still present a security risk.

In any case, you can see several corporations that are using AFS to solve global connectivity needs. Also visible are several organizations that have implemented multiple AFS cells rather than constructing a single, large cell. There is no ultimate limit to the size of a cell; in practice, the size of a cell depends more on internal politics and budgeting than it does on any technical limitations.

With their success, many of the original design team left CMU in 1989 to form Transarc Corporation, which packaged the old Andrew File System into "AFS." At this time, the commercial AFS product name is a registered trademark of Transarc and is no longer an acronym, so it is proper to speak of the AFS file system.

The first commercial release was version 2.3. As of 1997, the current release is 3.4a and, at the time of this writing, version 3.5 is being prepared for release at the beginning of 1998.

\[\[Footnote 1]]

As of 1997, about a thousand sites are running AFS. While this might seem small compared to the total number running NetWare or NFS, remember that the AFS sites typically have clients numbering an order of magnitude or two more. And, though AFS started off in academia, now over half of the site are commercial organizations. More significantly, because AFS sites can securely connect their file systems the global, Internet-based AFS shared file system has the greatest number of clients and servers, the largest amount of data, and the broadest geographic range of any on-line distributed file system. In fact, the T-shirts at the AFS user group symposiums declare the revolutionary phrase: "One world, One filesystem."

As for the stability of the firm, in 1994, Transarc was purchased by IBM. There appears to be no pressure from IBM to force Transarc to change its support model or favor one system vendor over another. Ports and enhancements are still scheduled according to the needs of the AFS market; IBM profits when Transarc makes money and not by forcing support for any particular piece of hardware. In general, this means that Sun and IBM servers and Sun desktops dominate the support schedule.

In the early 90s, Transarc - at that time still an independent company - collaborated with IBM, DEC, HP, and Bull in the solicitation by the Open Software Foundation for a new set of standard components of a distributed computing environment: a remote procedure call system from Apollo (now owned by HP), a domain name server and X.500 directory service from DEC, a distributed time service, and a major revision of AFS by Transarc.

These pieces of products were put together by the OSF and packaged as DCE and DFS. While AFS is a proprietary protocol maintained by a single company (though with extensive support through its collaboration with academic research institutions), DCE and DFS are maintained by a multivendor consortium. The long-term prospects for DCE/DFS are therefore potentially greater than AFS, but the larger installed base of AFS cells (as of 1997), its simpler administrative model, and lower initial costs make it an attractive alternative. Some of the similarities and differences between AFS and DFS are described in Chapter 12.

However, before trying to guess who will win the file system shakeout, note that there is no reason for any organization to support only a single protocol. In fact, for most, supporting only one protocol is impossible because of certain software packages which, for one reason or another, are wedded to some semantic behavior of a particular protocol. But, just as Fortran, COBOL, and C source code produce executables that can all run side-by-side on the same desktop, so too can multiple file systems be supported. Use of AFS at an organization does not imply a big-bang event, where one day file data comes from one set of disks and the next day it all comes from AFS servers. Distributed file systems can be complementary, especially during a migration phase, where different classes of data - home directories, development areas, production binaries - can be moved to the most appropriate location.

Certainly, NFS and NetWare are still being used at most sites that are using AFS. And other protocols will be used for other kinds of data; SQL for databases, HTTP for the Web, etc. As an organization employs these differing protocols, users and developers will vote with their data to find the best fit between their requirements and the underlying functionality. Chapter 11 includes some case studies that describe large and small sites which have found AFS to be indispensable and cost effective.

## Drawbacks <a href="#drawbacks" id="drawbacks"></a>

If all this sounds too good to be true, rest assured that there are several issues you should carefully examine before moving wholeheartedly into AFS. Most obviously, this is a proprietary technology. Only Transarc Corporation, produces fully supported ports to vendor hardware. Table 1-2 lists the ports currently available; this set of machines probably accounts for well over 90 percent of the installed base of UNIX workstations. Transarc has done a fair job of keeping up with new releases of these operating systems, but their porting schedule depends on satisfying their current customers first.

| Vendor                        | Operating System, Hardware                                           | AFS System Name   |
| ----------------------------- | -------------------------------------------------------------------- | ----------------- |
| Digital Equipment Corporation |                                                                      |                   |
|                               | Ultrix 4.3, DECstation 2100, 3100 or 5000 (single processor)         | pmax\_ul43 (3.4)  |
|                               | Ultrix 4.3a or 4.4, DECstation 2100, 3100 or 5000 (single processor) | pmax\_ul43a (3.4) |
|                               | Digital Unix 3.0, Alpha                                              | AXP               |
|                               | Digital Unix 2.0, Alpha                                              | AXP               |
|                               | Digital UNIX 3.2-3.2a, Alpha AXP                                     | alpha\_osf32      |
|                               | Digital UNIX 3.2c-3.2d, Alpha AXP (†)                                | alpha\_osf32c     |
|                               | Digital UNIX 4.0-4.0b, Alpha AXP                                     | alpha-dux40       |
| Hewlett-Packard               |                                                                      |                   |
|                               | HP-UX 9.0, 9.0.1, 9.0.3, 9000 Series 700 (†)                         | hp700\_ux90       |
|                               | HP-UX 9.0. 9.0.2, 9.0.4, 9000 Series 800 (†)                         | hp800\_ux90       |
|                               | HP-UX 10.01, 9000 Series 700 (†)                                     | hp700\_ux100      |
|                               | HP-UX 10.01, 9000 Series 800 (†)                                     | hp800\_ux100      |
|                               | HP-UX 10.10, 9000 Series 700 (†)                                     | hp700\_ux101      |
|                               | HP-UX 10.10, 9000 Series 800 (†)                                     | hp800\_ux101      |
|                               | HP-UX 10.20, 9000 Series 700 and 800 (†)                             | hp800\_ux102      |
| IBM                           |                                                                      |                   |
|                               | AIX 3.2, 3.2.1-3.2.5, RS/6000                                        | rs\_aix32         |
|                               | AIX 4.1.1, 4.1.3-5, RS/6000 (†)                                      | rs\_aix41         |
|                               | AIX 4.2, 4.2.1 RS/6000 (†)                                           | rs\_aix42         |
| NCR                           |                                                                      |                   |
|                               | NCR UNIX 2.0.2 and 3.0                                               | ncrx86\_30        |
| Silicon Graphics              |                                                                      |                   |
|                               | IRIX 5.2                                                             | sgi\_52 (3.4)     |
|                               | IRIX 5.3                                                             | sgi\_53           |
|                               | IRIX 6.1 (†)                                                         | sgi\_61           |
|                               | IRIX 6.2 (†)                                                         | sgi\_62           |
|                               | IRIX 6.3 (†)                                                         | sgi\_63           |
|                               | IRIX 6.4 (†)                                                         | sgi\_64           |
| Sun Microsystems              |                                                                      |                   |
|                               | SunOS 4.1.1-4.1.3, Sun 4 (non-SPARCstations)                         | sun4\_411 (3.4)   |
|                               | SunOS 4.1.1-4.1.3, Sun4c kernel                                      | sun4c\_411        |
|                               | SunOS 4.1.2, 4.1.3, 4.1.3\_U1, Sun4m kernel (†)                      | sun4m\_412        |
|                               | SunOS 5.3, Sun 4 (non-SPARCstations)                                 | sun4\_53 (3.4)    |
|                               | SunOS 5.3, Sun4c kernel (†)                                          | sun4c\_53         |
|                               | SunOS 5.3, Sun4m kernel (†)                                          | sun4m\_53         |
|                               | SunOS 5.4, Sun 4 (non-SPARCstations)                                 | sun4\_54 (3.4)    |
|                               | SunOS 5.4, Sun4c kernel (†)                                          | sun4c\_54         |
|                               | SunOS 5.4, Sun4m kernel (†)                                          | sun4m\_54         |
|                               | SunOS 5.5-5.5.1, all kernels (†)                                     | sun4x\_55         |
|                               | SunOS 5.6, all kernels (†)                                           | sun4x\_56         |
| Microsoft Windows             |                                                                      |                   |
|                               | NT 3.51, 4.0, x86 only                                               | i86\_nt           |

&#x20;† - including multiprocessors

&#x20;(3.4) - AFS 3.4 only; no new releases planned

&#x20;Table 1-2: Supported Hardware/OS Ports of AFS

You might note in this list that the internal Transarc system name does not always coincide with the operating system release number, as with Sun's operating system 4.1.3, which is supported by AFS version sun4m\_412. The reason is simply that Transarc doesn't always have to recompile their binaries to support a new operating system release. Check with Transarc if your system is not listed: some earlier versions are available. Naturally, they'll port it to anything for the right price.

On the other hand, one reason for the continued improvements to the system is that Transarc makes the source available for a very reasonable price. Many sites, educational institutions especially, have been able to go over the code and find exactly where problems occur with new operating system releases or other resource contentions. And ports to unsupported systems, such as NetBSD or Linux on Intel™-based hardware, have been produced by other individuals and made available from Transarc's Web site. (In this respect, the AFS community feels much more like an open standard consortium than, say, DFS.) And for systems that are still unsupported, it is possible for a client to use NFS, NetWare, or other protocols to access a gateway machine that is a client of AFS, thereby providing access to the AFS file namespace.

Another issue is the fundamental architecture: CMU wanted to provide centralized file services for a large, distributed campus, and AFS is the result. If your organization cannot provide management support for centralized file services or if your particular division is extremely small, then AFS just won't work too well. For a department with a half-dozen workstations and a single file server, AFS is probably overkill. Of course, as the department inevitably grows, AFS becomes more attractive. And some day, someone will point out that all of the individual departments are reinventing the same administrative wheel as maps and disks and software and users are installed and maintained on dozens of independently managed servers. If an organization is structured around small-scale, decentralized servers, AFS in and of itself won't reduce the hardware and administrative overhead.

Indeed, effective use of AFS requires a lot of reengineering not simply of technology but of the enterprise. While the system comprises distributed components and takes advantage of the price, performance, and functionality of modern computing, it seeks to recentralize and coordinate file storage. For example, the system encourages a small number of AFS administrative domains, potentially just one. In this domain, there is a single repository of security information. Rather than have multiple, independently managed servers each with their own set of users, this single domain will have just a single set of users - all the members of an enterprise, each uniquely identified. Implementing this is mostly a political question.

For organizations that wish to use AFS to provide good centralized file services, the biggest risk is the education of the operations staff. It will do no good to centralize file services if only one or two people know how the system works. Nowadays, it is customary to hire administrators who are already fully versed in NFS or NetWare. To get the best results from AFS requires systematic training, probably from Transarc, for all of your staff. And the education doesn't stop there, for users will have to have a smattering of AFS knowledge, such as the issues described in Chapter 6 - access controls, authentication, group management, and those few commands (such as disk space usage) that don't work as expected in the AFS file namespace.

The most contentious aspect of AFS has to do with its imperfect mapping of local file behaviors, expected by certain applications, to its distributed system interfaces. Naturally, as applications have been developed over the years, many of them have been optimized for the characteristics of the local operating and disk system. For these few applications, either local or other distributed file systems may have to be supported for some time.

Certainly, some application data files should be stored in specialized file systems; as one good example, high-powered databases will still need their own raw disk surfaces and server hardware with which to implement their own distribution and replication architecture. For another example, certain UNIX files, such as block or character device files and named pipes, don't have established semantics in a distributed world and are unsupported in AFS; these too, remain wedded to their local machine.

And it is not difficult to find home-built or third-party applications that depend on looser, UNIX-style authentication models that simply do not translate well into a strictly authenticated Kerberos system. Many jobs in a UNIX environment are started up by `cron` or `rsh` and need to write file data. If those files are to be managed by AFS, their access will be controlled by Kerberos-authenticated identities, identities which standard UNIX tools have trouble obtaining or sometimes forget. Lack of support for these applications is rarely a show-stopper, but it is often an important issue; many power users are especially bothered by this problem as they've come to accept the weak distributed security model of both UNIX and PC systems. With the strong, mutual authentication guarantees of Kerberos, old mechanisms based on untrustworthy desktops are no longer valid.

Though Transarc is in the file storage business, their backup and archiving system is a little rudimentary though comprehensive. It certainly does the job of reliably saving and restoring file data and internal AFS meta-data when and as needed. But it is certainly not as flexible or graphically enticing as most of the third-party backup systems available. By now, several companies have begun to integrate AFS support with their archive products - both to back up the file data and also the AFS volume structures. Besides these, several other techniques have been developed by the AFS community to use Transarc's tools or home-grown hacks to provide additional functionality; Chapter 7 discusses some of these.

Because AFS is complex and its installation needs to be optimized for each enterprise, much like any other database system, Transarc does not supply a free demo of AFS downloadable from the Net. It may sound as though Transarc would miss sales opportunities, but given the complexity of the system, it's all too easy for a customer to miss the point and reject AFS out of hand. (Of course, this book should help to some extent to describe what it is that AFS can do for an organization and will certainly provide tips on how to use the system effectively.) But so far, Transarc wants to be in the loop on the sales call to provide a human point-of-contact and to guide potential customers to success.

All of these issues are significant and should be examined when you investigate AFS and Transarc. But studies have shown and extensive use of the system has demonstrated that AFS is a powerful, successful product that can provide demonstrably better distributed file system performance and efficiencies to many organizations.

To those who believe that AFS is too expensive in a world where disk drives become cheaper every month: Understand that other distributed file systems bundled with machines may be free, but the on-going administration costs quickly outstrip any imagined advantages. The costs of file storage are not simply the dollars spent on hardware but on the investments needed to control the ever increasing amount of data needed by programs, data, and developers. Disks are cheap, but the data stored on them is of paramount importance to most organizations and therefore justifies the use of appropriate technologies to keep the system usable. Certainly the risks of data loss and unavailability are far more important than choosing protocols based solely on purchase price.

## Other Sources Of Information <a href="#other-sources-of-information" id="other-sources-of-information"></a>

The best source of the information on AFS is Transarc itself:

```
        Transarc, Corporation
        The Gulf Tower
        707 Grant Street
        Pittsburgh, PA 15219 USA
 
        phone: 412-338-4400
        fax: 412-338-4404
        e-mail: information@transarc.com or afs-sales@transarc.com
        Web: http://www.transarc.com/
```

A complete set of hardcopy documentation is shipped with a product purchase. The latest release of the system, version 3.4a, comes with the following manuals:

* AFS System Administration Guide - Detailed chapters covering the architecture, behavior, common practices and use of all of the AFS commands. Not quite exhaustive; some information is to be found only in the Reference Guide.
* AFS Command Reference Guide - An alphabetical listing of each command and every option.
* AFS Installation Guide - How to get AFS up and running at your site. Somewhat confusing for the first-time AFS administrator; contains platform-specific sections in between architecture-independent sections.
* AFS User's Guide - Information on AFS needed to help user's navigate the enhanced features of the file system.
* Release Notes - Not only lists the latest fixes and workarounds but also describes new functionality not included in the main documentation.

Transarc is converting all of their documentation to the Web HTML standard, shipping on-line copies with their product and even permitting viewing of the available text from their Web site. The AFS manual set should be done, as we say, real soon now.

Source code for the entire system is available at a reasonable price. This is not quite the luxury it seems: When you need to write custom management tools that interface with the AFS kernel modules or client/server RPCs, using the source is the best way to quickly complete the job. While the documentation tries to describe the interfaces, the examples are difficult to follow - it is wiser and almost essential to cut'n'paste code from Transarc's own utilities to quickly produce your own specialized programs.

While the source code is not free, Transarc has permitted certain client binaries created from it to be distributed. Over the years, enterprising individuals have ported the AFS client code to some of the other UNIX ports now available. Currently, you can find client executables for the Linux and NetBSD UNIX variants available for free to any AFS site; though Transarc appears to encourage client proliferation, they'll still want to charge you for their AFS servers.

There is quite of bit of free code and other documentation in Transarc's public area of their FTP site: `ftp://ftp.transarc.com/pub`

Here, a variety of subdirectories with information, source code, and scripts help you to understand and administer AFS. Some programs are almost essential to investigating odd client behavior; others have been devised and contributed by other AFS sites and administrators.

Transarc also publishes a technical journal that it mails to each site. Besides highlighting upcoming product features, it usually contains a description of a customer's AFS cell configuration. Such examples are useful in confirming certain cell designs you may have questions about.

Questions and comments regarding AFS can be directed to a public mailing list for general discussion of problems and solutions. When sent to `info-afs@transarc.com`, the message goes to all subscribers. Currently, the list processes just one or two mail messages a day and the postings are usually informative. You can subscribe to this mailing list by sending a message to `majordomo@transarc.com` with the sentence `subscribe info-afs` as the body of the message. Getting off the list is similar: just insert "unsubscribe" in the body. Make sure you send the message to `majordomo` and not to the list `info-afs` as a whole.

For several years Transarc hosted an AFS User's Group conference. With the expansion of the Transarc product line to incorporate DCE, DFS, and the Encina™ transaction monitor, the conference focus has been broadened to embrace these DCE technologies as well as AFS. Held once a year, usually in the early spring, this conference brings together some of the DCE and DFS implementors from various vendors, as well as many third-party software publishers and commercial users of the system.

And of course, other information is available on the Internet, including a Frequently Asked Questions document, to which there is a pointer at the Transarc Web site.

## Summary <a href="#summary" id="summary"></a>

This chapter presented an overview of the benefits of AFS. Actually achieving these benefits depends on a careful implementation plan. Such a plan must not only construct a technological infrastructure that correctly supplies AFS's advantages to the desktop, but also must incorporate the computing culture at any given organization.

Typically, sites that embark on a migration of file services into AFS undergo growing pains as they invent new administration practices and unlearn habits caused by the vagaries of their previous infrastructure. Once the migration is in full swing, a critical point is quickly reached where the benefits of the new system are self-evident. For users, the change amounts to learning to navigate a slightly different file-naming convention; for administrators, the ease of use and maintainability of AFS makes them wonder why they took so long to make the move in the first place.

As the details of AFS are described, we will often compare AFS to NFS or NetWare. The point is not to start a war of words over which is better but simply to discuss the different implementations and how those designs reflect the initial goals of the products. There's no denying that NFS is the best NFS around. And as far as it goes, it is functional and extremely useful. But it is not the only distributed file system available.

The next chapter provides a detailed look at the technology and protocol that make up AFS clients and servers. After that, the following chapters return to a somewhat more general view of administration practices and use of the system.

\[\[Footnote 1]] By the way, the SCO Corporation's UNIX operating system, OpenServer™, also has an AFS file system: this is the Acer© Fast File System and is a local disk file system only. Also, the Apple© Macintosh© file system and the Amiga File System are occasionally referred to as AFS. Don't confuse these file systems with Transarc's distributed file system.
