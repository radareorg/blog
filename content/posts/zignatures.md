+++
date = "2014-11-05T22:16:46+01:00"
draft = false
title = "Zignatures"
slug = "zignatures"
aliases = [
	"zignatures"
]
+++
by http://twitter.com/j0sm1
modified by @Obaied

In this blog post, we are going to show a simple example of the radare2 “zignatures” functionality. To manage “zignatures” in radare2, the only thing you have to do is type ‘z’ in the radare console. Here you can get more info on this 'z' command:

    r2console> z?
    |Usage: z[abcp/*-] [arg]Zignatures
    | z                 show status of zignatures
    | z*                display all zignatures
    | z-prefix          unload zignatures with corresponding prefix
    | z-*               unload all zignatures
    | z/[ini] [end]     search zignatures between these regions
    | za ...            define new zignature for analysis
    | zb name bytes     define zignature for bytes
    | zc @ fcn.foo      flag signature if matching (.zc@@fcn)
    | zf name fmt       define function zignature (fast/slow, args, types)
    | zg prefix [file]  generate signature for current file
    | zh name bytes     define function header zignature
    | zp prefix         define prefix for following zignatures
    | zp                display current prefix
    | zp-               unset prefix
    | NOTE:             bytes can contain '.' (dots) to specify a binary mask

To show this functionality, we are going to use a simple C server. You can see its code here:

    /* A simple server listening on TCP port 4001
       from http://www.linuxhowtos.org/data/6/server.c, modified by Sam Bowne
    */
    #include <netinet/in.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <sys/socket.h>
    #include <sys/types.h>
    #include <unistd.h>

    int copier(char *str) {
        char buffer[1024];
        strcpy(buffer, str);
        return 0;
    }

    void error(const char *msg) {
        perror(msg);
        exit(1);
    }

    int main(int argc, char *argv[]) {
        int sockfd, newsockfd, portno;
        socklen_t clilen;
        char buffer[4096], reply[5100];
        struct sockaddr_in serv_addr, cli_addr;
        int n;
        sockfd = socket(AF_INET, SOCK_STREAM, 0);
        if (sockfd < 0) error("ERROR opening socket");
        bzero((char *)&serv_addr, sizeof(serv_addr));
        portno = 4001;
        serv_addr.sin_family = AF_INET;
        serv_addr.sin_addr.s_addr = INADDR_ANY;
        serv_addr.sin_port = htons(portno);
        if (bind(sockfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0)
            error("ERROR on binding");
        listen(sockfd, 5);
        clilen = sizeof(cli_addr);
        newsockfd = accept(sockfd, (struct sockaddr *)&cli_addr, &clilen);
        if (newsockfd < 0) error("ERROR on accept");
        while (1) {
            n = write(newsockfd, "Welcome to my server!  Type in a message!\n", 43);
            bzero(buffer, 4096);
            n = read(newsockfd, buffer, 4095);
            if (n < 0) error("ERROR reading from socket");

            copier(buffer);

            printf("Here is the message: %s\n", buffer);
            strcpy(reply, "I got this message: ");
            strcat(reply, buffer);
            n = write(newsockfd, reply, strlen(reply));
            if (n < 0) error("ERROR writing to socket");
        }
        close(newsockfd);
        close(sockfd);
        return 0;
    }

First, we are going to compile this example statically and without stripping the compiled code:

    $ clang server.c -static -o server
    $ r2 server
    ## We can check if the binary is stripped with this r2 command
    > i~strip
    strip false

On the other hand, we compiled statically this same example with name server2, and this time we also executed the strip command to delete information from this binary (symbols, etc.):

    $ clang server.c -static -o server2
    $ strip server2
    $ r2 server2

We can check if the binary is stripped with this r2 command

    > i~strip
    $ strip true

We can see that the symbols are stripped when loading the stripped binary with r2

    $ r2 server2
    > aa
    > afl

    0x00401a30    1 47           entry0
    0x004020c0  101 1835         fcn.004020c0


Now, we create the “zignatures” from our static and unstripped binary (server). After that, we will apply this “zignatures” over the stripped binary (server2).

We create “zignature” file with command ‘zg’ and `zos`.

    $ r2 server
    > aa
    > zg            # Generate zignatures from the binary
      ...
      ...
      ...
      generated zignatures: 845
    > zos zigs.z    # Save the zignatures to a file zigs.z

It’s time to open our stripped binary and watch which information is available. Load stripped binary with auto analysis:

    $ r2 server2
    > aa
    > zo zigs.z     # Load zigs.z as a zignatures file
    > z/            # Scan the binary in search for flags that correspond to the found zignatures

To show the zignatures loaded use the 'z' command.

    > z
    Loaded 4702 signatures
      4702 byte signatures
      0 head signatures
      0 func signatures

Now, we are able to identify the stripped binary from the generated flags. For example, let's see if we can identify the `copier()` method


    > f~sym
      0x00401000 27 sign.bytes.sym._IO_list_resetlock_0
      0x004010b0 16 sign.bytes.sym.__assert_fail_base.cold.0_0
      0x00401000 27 sign.bytes.sym._init_0
      0x004010c5 500 sign.bytes.sym.abort_0
      0x004012e7 333 sign.bytes.sym._IO_new_fclose.cold.0_0
      0x00401434 246 sign.bytes.sym._IO_fputs.cold.0_0
      0x00401580 16 sign.bytes.sym.__cxa_atexit_0
      0x00401a70 33 sign.bytes.sym.deregister_tm_clones_0
    > f~copier
      0x00401b60 127 sign.bytes.sym.copier_0
    ...
    ...
    ...
    > pd 10 @ sign.bytes.sym.copier_0
    Free fake stack
                ;-- sign.bytes.sym.copier_0:
                0x00401b60      55             push rbp
                0x00401b61      4889e5         mov rbp, rsp
                0x00401b64      4881ec200400.  sub rsp, 0x420
                0x00401b6b      64488b042528.  mov rax, qword fs:[0x28]    ; [0x28:8]=0 ; '('
                0x00401b74      488945f8       mov qword [rbp - 8], rax
                0x00401b78      4889bde8fbff.  mov qword [rbp - 0x418], rdi
                0x00401b7f      488bb5e8fbff.  mov rsi, qword [rbp - 0x418]
                0x00401b86      488dbdf0fbff.  lea rdi, [rbp - 0x410]


We can trace the execution of the code and identify that the loaded zignature xrefs in the Main function.

In this post we have seen how zignatures can help us to make the analysis of stripped binaries easier.
