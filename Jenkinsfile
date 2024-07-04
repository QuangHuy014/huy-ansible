- name: ec2instance-playbook
  hosts: host
  become: yes
  become_user: root
  become_method: sudo
  tasks:
    - name: update
      apt:
        upgrade: yes
        update_cache: yes
        cache_valid_time: 86400

    - name: Install required system packages
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - python3-pip
          - virtualenv
          - python3-setuptools
        state: latest
        update_cache: true

    - name: Download Docker GPG key
      command: curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /tmp/docker.gpg

    - name: Add Docker GPG key
      command: apt-key add /tmp/docker.gpg

    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu focal stable
        state: present

    - name: Update apt and install docker-ce
      apt:
        name: docker-ce
        state: latest
        update_cache: true

    - name: Create a docker network
      docker_network:
        name: test

    - name: Re-create a MySQL container
      docker_container:
        name: quanghuy-mysql
        image: mysql:8.0
        networks:
          - name: test
            aliases:
              - test
        env:
          MYSQL_ROOT_PASSWORD: 12345
          MYSQL_DATABASE: Planny
        detach: true
        state: started
        recreate: yes
        exposed_ports:
          - 3306
        pull: true
        comparisons:
          image: strict

    - name: Copy file with owner and permissions
      ansible.builtin.copy:
        src: ./script
        dest: /home/ubuntu/script
        owner: ubuntu
        group: ubuntu

    - name: SLEEP now !!!
      shell: sleep 15 && sudo docker exec -i quanghuy-mysql mysql --user=root --password=12345 < /home/ubuntu/script

    - name: Re-create a Spring container
      docker_container:
        name: quanghuy-springboot
        image: huynq201104/springboot
        networks:
          - name: test
            aliases:
              - test
        state: started
        recreate: yes
        exposed_ports:
          - 8080
        detach: true
        published_ports:
          - 8080:8080
        pull: true
        comparisons:
          image: strict

    - name: Prune everything
      community.docker.docker_prune:
        containers: true
        images: true
