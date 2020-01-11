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
    