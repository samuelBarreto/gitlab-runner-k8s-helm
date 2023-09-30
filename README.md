# Instalando o GitLab Runner usando o Helm Chart

## Pré-requisitos
    - helm
    - A API do seu servidor GitLab pode ser acessada a partir do cluster.
    - Kubernetes 1.4+ com APIs Beta habilitadas.
    - A CLI kubectl instalada localmente e autenticada para o cluster.

'''
    Adicione o repositório GitLab Helm:

            helm repo add gitlab https://charts.gitlab.io

'''''

  ## criar namespace gitlab

         --- kubectl create namespace gitlab 

### criar arquivo values.yaml


    Configuração necessária
    Para que o GitLab Runner funcione, seu arquivo de configuração deve especificar o seguinte:

        gitlabUrl: O URL completo do servidor GitLab para registrar o executor. Por exemplo,, https://gitlab.example.com.
        rbac: { enable: true }: Crie regras RBAC para o GitLab Runner criar pods para executar trabalhos. Se você tiver uma serviceAccount existente que prefere usar, você também deve definir rbac: { serviceAccountName: "SERVICE_ACCOUNT_NAME" }. Para obter mais informações sobre as permissões mínimas necessárias para o serviceAccount

    '''' FILE.YAML '''

    ### Aqui 

    Alguns detalhes cruciais que precisam de sua atenção são:

    gitlabUrl: substitua pelo URL da sua instância do GitLab. Se você usa Gitlab.com, o valor deve ser https://gitlab.com
    runnerRegistrationToken: pode ser encontrado no menu CI/CD > Runner
    privilegiado: defina verdadeiro se você usar docker-in-docker (dind)

    ''''
                gitlabUrl: https://gitlab.example.com/

                runnerRegistrationToken: "xxx"

                concurrent: 10

                checkInterval: 30

                rbac:       
                create: true
                rules:
                    - apiGroups: [""]
                    resources: ["pods"]
                    verbs: ["list", "get", "watch", "create", "delete"]
                    - apiGroups: [""]
                    resources: ["pods/exec"]
                    verbs: ["create"]
                    - apiGroups: [""]
                    resources: ["pods/log"]
                    verbs: ["get"]
                    - apiGroups: [""]
                    resources: ["pods/attach"]
                    verbs: ["list", "get", "create", "delete", "update"]
                    - apiGroups: [""]
                    resources: ["secrets"]
                    verbs: ["list", "get", "create", "delete", "update"]      
                    - apiGroups: [""]
                    resources: ["configmaps"]
                    verbs: ["list", "get", "create", "delete", "update"]     

                runners:
                privileged: true
                
                config: |
                    [[runners]]
                    [runners.kubernetes]
                        namespace = "gitlab"
                        tls_verify = false
                        image = "docker:19"
                        privileged = true
    ''

instalacao com helm
''''
    helm install gitlab-runner gitlab/gitlab-runner -f values.yaml --namespace gitlab
''

verificar instalacao
''''
    kubectl -n gitlab get deployment

    NAME            READY   UP-TO-DATE   AVAILABLE   AGE
    gitlab-runner   1/1     1            1           42d
''

