# defa_part2_services
Services from the part2 materials https://devopswithdocker.com/part2/

# clone the repository

    git@github.com:vaaPo/defa_part2_services.git

# whoami from command line

    docker run -d -p 8000:8000 jwilder/whoami 
## kill the local service blocking port 8000

    docker: Error response from daemon: driver failed programming external connectivity on endpoint gifted_black (b00628ecd5ec1120e4e32925cabb4213ad918f879f30a4a0a87a61f470dbccd2): Error starting userland proxy: listen tcp 0.0.0.0:8000: bind: address already in use.

    sudo /etc/init.d/webfs stop

## curl whoami

    curl http://localhost:8000

