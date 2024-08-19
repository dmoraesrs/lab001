---
- name: Configurar servidor com Nginx, MariaDB e atualizar SO
  hosts: all
  become: yes

  tasks:
    - name: Aplicar as últimas atualizações do sistema operacional
      apt:
        upgrade: dist
        update_cache: yes

    - name: Instalar Nginx
      apt:
        name: nginx
        state: present

    - name: Criar página de bem-vindo
      copy:
        dest: /var/www/html/index.html
        content: |
          <!DOCTYPE html>
          <html lang="en">
          <head>
              <meta charset="UTF-8">
              <meta name="viewport" content="width=device-width, initial-scale=1.0">
              <title>Bem-vindo à Tilabs</title>
          </head>
          <body>
              <h1>Bem-vindo à Tilabs</h1>
              <p>Esta é a página de boas-vindas do Nginx configurado via Ansible.</p>
          </body>
          </html>

    - name: Garantir que o Nginx esteja ativo e habilitado no boot
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Instalar MariaDB
      apt:
        name: mariadb-server
        state: present

    - name: Garantir que o MariaDB esteja ativo e habilitado no boot
      service:
        name: mariadb
        state: started
        enabled: yes

    - name: Garantir que o firewall permite o tráfego HTTP e HTTPS
      ufw:
        rule: allow
        name: "{{ item }}"
      loop:
        - 'Nginx HTTP'
        - 'Nginx HTTPS'

    - name: Reiniciar o Nginx para aplicar as últimas configurações
      service:
        name: nginx
        state: restarted

    - name: Reiniciar o MariaDB para garantir a aplicação das configurações
      service:
        name: mariadb
        state: restarted
