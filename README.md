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

