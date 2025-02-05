DOCKER-ROLE
=========

this role setups the docker on ubuntu

Role Variables
--------------

docker_version: 27.5.1 => the version of docker you want to install 
docker_url: https://download.docker.com/linux/static/stable/x86_64/docker  => the binary url
docker_dest: /opt => download directory of binary file
docker_user: ubuntu => user to add in docker group

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------
Name: luffy
Email: luffysenpai661@gmail.com
