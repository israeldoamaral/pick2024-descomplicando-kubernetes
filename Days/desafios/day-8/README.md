Desafio Nginx com HTTPS no Kubernetes

🔬 Neste desafio, você precisa configurar o Nginx com HTTPS em um cluster Kubernetes. Isso envolve a criação e manipulação de Secrets e ConfigMaps. Seu objetivo é demonstrar sua compreensão dos conceitos e habilidades práticas em Kubernetes.

Instruções
IMPORTANTE: Todos os arquivos devem ser criados no diretório /root/manifests, inclusive o arquivo nginx.conf e o certificado e a chave privada.

🏗️ Crie um Secret no Kubernetes que contenha o certificado e a chave privada para o HTTPS. Verifique se o Secret foi criado corretamente e se o tipo está correto.

🎯 Crie um ConfigMap com a configuração do Nginx. Certifique-se de que a configuração está correta e de que o ConfigMap está corretamente configurado.

💼 Verifique se o Nginx está rodando e se seu status é "Running".

🕵️‍♀️ Certifique-se de que o HTTPS está funcionando corretamente. Para fazer isso, você pode executar curl -k https://localhost:<PORTA>.

📝 Verifique o certificado do Nginx e certifique-se de que está correto.

💡 Verifique se o Service é do tipo NodePort na porta 32400 e está redirecionando para a porta 443 do container.

📝 Verifique se o Nginx está rodando na porta 443.

🎯 O nome do Pod precisa ser nginx-https.

🎯 O nome do Service precisa ser nginx-service.

🎯 O nome do ConfigMap precisa ser nginx-config.

🎯 O nome do Secret precisa ser nginx-secret.

🎯 O certificado e a chave privada precisam estar no diretório /root/manifests.

🎯 O arquivo de configuração do Nginx precisa estar no diretório /root/manifests.

🎯 O arquivo de configuração do Nginx precisa se chamar nginx.conf.

🎯 O certificado precisa se chamar nginx.crt.

🎯 A chave privada precisa se chamar nginx.key.

🎯 O manifesto do Pod precisa se chamar nginx-https-pod.yaml.

🚀 Use as guias "Terminal", "Editor" e "Navegador" para concluir este desafio. Após concluir as instruções, pressione o botão Check para verificar se você solucionou o desafio corretamente.