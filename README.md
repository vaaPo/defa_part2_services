# defa_part2_services
Services from the part2 materials https://devopswithdocker.com/part2/

# clone the repository

    git@github.com:vaaPo/defa_part2_services.git

# files

# whoami from command line

    docker run -d -p 8000:8000 jwilder/whoami 
## kill the local service blocking port 8000

    docker: Error response from daemon: driver failed programming external connectivity on endpoint gifted_black (b00628ecd5ec1120e4e32925cabb4213ad918f879f30a4a0a87a61f470dbccd2): Error starting userland proxy: listen tcp 0.0.0.0:8000: bind: address already in use.

    sudo /etc/init.d/webfs stop

## curl whoami

    curl http://localhost:8000

# whoami/docker-compose.yml

    docker-compose up -d 
    docker-compose stop

# port clash
    docker-compose stop
    docker-compose up --scale whoami=3 
    docker-compose stop
    docker-compose up -d --scale whoami=3 
    docker-compose port --index 1 whoami 8000 
    docker-compose port --index 2 whoami 8000 
    docker-compose port --index 3 whoami 8000 

## fix ports by not nailing down the port 8000
    https://docs.docker.com/compose/reference/port/

    ### port and expose
    
    expose would expose "internal" ports to linked services, not to host

    port exposes ports host_port:container_port but dont use ports under 60, it wont work - so use strings like "8000:9000"
    also port wont work with network_mode: host
    or use long syntax:
        port
            - target: 80                        ## the container_port
              published: 8080                   ## public port - host_port
              protocol: tcp                     ## tcp or udp
              mode: host

    strange thing is that if port is given only one port, then it is the container_port and the host port gets dynamically allocated as next free?


    Print the public port for a port binding.

    Usage: port [options] SERVICE PRIVATE_PORT

    Options:
        --protocol=proto  tcp or udp [default: tcp]
        --index=index     index of the container if there are multiple
                        instances of a service [default: 1]

# PROXIED whoami proxied_whoami

    Load balancer based on nginx-proxy https://github.com/jwilder/nginx-proxy 
     that configures nginx from docker daemon as containers are started and stopped.
    - ports removed from whoami
    - socket volume for nginx in read-only

    $ docker-compose up -d --scale whoami=3 
    Creating network "proxiedwhoami_default" with the default driver
    Creating proxiedwhoami_whoami_1 ... 
    Creating proxiedwhoami_whoami_2 ... 
    Creating proxiedwhoami_whoami_3 ... 
    Creating proxiedwhoami_proxy_1 ... 
    Creating proxiedwhoami_whoami_1
    Creating proxiedwhoami_whoami_1 ... done
    Creating proxiedwhoami_whoami_2 ... done
    Creating proxiedwhoami_whoami_3 ... done
    $ curl localhost:80 
    <html>
    <head><title>503 Service Temporarily Unavailable</title></head>
    <body>
    <center><h1>503 Service Temporarily Unavailable</h1></center>
    <hr><center>nginx/1.17.5</center>
    </body>
    </html>

    ## but nginx doesnt know services
        see https://github.com/jwilder/whoami/blob/master/Dockerfile#L9
        The nginx-proxy works with two environment variables: VIRTUAL_HOST and VIRTUAL_PORT. 
        VIRTUAL_PORT is not needed if the service has EXPOSE in it’s docker image.

        ### colasloth - tweaking
            See https://colasloth.github.io/
            it simply states to use colasloth type of fqdn instead of localhosts. Even when colasloth is localhost.
            $ ping whoami.colasloth.com
            PING whoami.colasloth.com (127.0.0.1) 56(84) bytes of data.
            64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.054 ms
            64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.118 ms
            ^C
            --- whoami.colasloth.com ping statistics ---
            2 packets transmitted, 2 received, 0% packet loss, time 1019ms
            rtt min/avg/max/mdev = 0.054/0.086/0.118/0.032 ms
        
    docker-compose up -d --scale whoami=3
    $ docker-compose up -d --scale whoami=3 
    /usr/lib/python2.7/dist-packages/requests/__init__.py:80: RequestsDependencyWarning: urllib3 (1.22) or chardet (2.1.1) doesn't match a supported version!
    RequestsDependencyWarning)
    Recreating proxiedwhoami_whoami_1 ... 
    Recreating proxiedwhoami_whoami_2 ... 
    Recreating proxiedwhoami_whoami_3 ... 
    Starting proxiedwhoami_proxy_1 ... 
    Starting proxiedwhoami_proxy_1
    Recreating proxiedwhoami_whoami_1 ... done
    Recreating proxiedwhoami_whoami_2 ... done
    Recreating proxiedwhoami_whoami_3 ... done
    $ docker container ps
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                NAMES
    85ca25f606f5        jwilder/whoami        "/app/http"              9 seconds ago       Up 6 seconds        8000/tcp             proxiedwhoami_whoami_2
    d858b8752107        jwilder/whoami        "/app/http"              9 seconds ago       Up 7 seconds        8000/tcp             proxiedwhoami_whoami_1
    22e68b1b26cf        jwilder/whoami        "/app/http"              9 seconds ago       Up 5 seconds        8000/tcp             proxiedwhoami_whoami_3
    2346062a0dae        jwilder/nginx-proxy   "/app/docker-entrypo…"   8 minutes ago       Up 7 seconds        0.0.0.0:80->80/tcp   proxiedwhoami_proxy_1

    paavo@silber:~/defa/courseware/part2/defa_part2_services/proxied_whoami$ curl whoami.colasloth.com 
    I'm 85ca25f606f5
    paavo@silber:~/defa/courseware/part2/defa_part2_services/proxied_whoami$ curl whoami.colasloth.com 
    I'm d858b8752107
    paavo@silber:~/defa/courseware/part2/defa_part2_services/proxied_whoami$ curl whoami.colasloth.com 
    I'm 22e68b1b26cf
    
# Add more content to proxied services on the fly

    $ echo "hello" > hello.html
    $ echo "world" > world.html 
    Fix the docker-compose.yml to contain hello and world services.
    $ docker-compose up -d --scale whoami=3 
    $ curl hello.colasloth.com 
    hello

    $ echo "hello, this really rocks" > hello.html
    $ curl hello.colasloth.com 
    hello, this really rocks

# redmine
    https://www.redmine.org/
    https://www.adminer.org/
    https://hub.docker.com/_/redmine
    https://hub.docker.com/_/postgres
    https://hub.docker.com/_/adminer

    $ docker-compose up
    $ docker inspect db_redmine | grep -A 5 Mounts
            "Mounts": [
            {
                "Type": "volume",
                "Name": "5a475ce38849128b22014c96965ae3c99fef7b7c476bb83b3243562d06f6ebad",
                "Source": "/var/lib/docker/volumes/5a475ce38849128b22014c96965ae3c99fef7b7c476bb83b3243562d06f6ebad/_data",
                "Destination": "/var/lib/postgresql/data",
    $ docker volume ls
    $ docker volume ls | grep 6ebad
    local               5a475ce38849128b22014c96965ae3c99fef7b7c476bb83b3243562d06f6ebad
    $ docker volume prune
    WARNING! This will remove all local volumes not used by at least one container.
    Are you sure you want to continue? [y/N] 
    ....
    Total reclaimed space: 442.7MB
    $ docker volume ls 
    DRIVER              VOLUME NAME
    local               0e5faf03fe5c2a64e3dd407f54f49664b599a200c543d163b50d32d4ebf51d20
    local               4bcb28899a0fb139205d23a5b30ed0bb27e7421dbda88b7d4077158ffa12d110
    local               4288bfda548f6cdab8f86553332f2394c67072547da986e08e1e85c93be96c28
    local               7520a0172f42ea76568f5d53d2bacea9dfe0cb67e8cea952c78ca2a7c3721457
    local               62047b84cda9f7eb7892f1da661c53160f993c27778cb7909c9112a23935ece0
    local               acdf88938531c5faf7fd03180053f10f9872f2ed095b3127183540742a9f8835
    local               af3e1b27338bd280de36dc541e083938cb00d99aaee9a95d35aedfd853e87c05
    local               cb55803662002383125d55c5858351944c70a1b3ae7c460826fa4366bbfd138a
    local               ccbed1de47c4739b542e6b8f1a768429cd203a68a35a0adfcef9030daf39c965
    local               d9c398babdfec817333fbdf088fdb8734533a3304e465c5faf2059ce712a6e20
    local               d44df5a229b80836493b6b2f3aea9a5c3153ba8130e2414d096fc9f26f8770df
    local               da5dc088b84043ee45edbb3fe17a07e915556737f62131a168c7b26251fb36dd
    local               dist
    local               ea3948e27991df7d5440f5e2e1942d74a4a65c97d592039cc01ea4e8cf5d38ae
    local               f251cc0b4fda6282507ba35a1ed7540b95e3d9d3b763dbabe397a84a87c7e01b
    local               redmine_database
    local               src

    After adding 2 volumes amd redmine:
    $ docker volume ls | grep red0
    local               redmine_database
    local               redmine_files

    Surf to http://localhost:9999/
    make changes

    check if different
    $ docker diff $(docker-compose ps -q redmine) 

    Add adminer to docker-compose.yml
    Access adminer via: http://localhost:8083/
    
    ![see screenshotfile](./redmine/adminer-for-redmine.png?raw=true "./redmine/adminer-for-redmine.png")

