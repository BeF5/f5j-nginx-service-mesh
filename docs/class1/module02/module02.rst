NGINX Ingress Controller(NIC) 環境のセットアップ
####


| 以下の手順に従ってNGINX Ingress Controllerのイメージを作成します  
| 参考： `Installation with Manifests <https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/>`__

1. NGINX Ingress Controller(NIC)セットアップ
====

NGINX Service Mesh で NGINX Ingress Controller(NIC)を利用するため、
以下の手順に従ってNICのデプロイを完了してください

`NGINX Ingress Controller(NIC) 環境のセットアップ <https://f5j-nginx-ingress-controller-lab1.readthedocs.io/en/latest/class1/module2/module2.html>`__


``5. NGINX Ingress Controllerの実行`` は 以下の内容を参考に実行してください。
.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/deployments
  cp deployment/nginx-plus-ingress.yaml deployment/nginx-plus-ingress-sm.yaml
  vi deployment/nginx-plus-ingress-sm.yaml


.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/deployments
  cp deployment/nginx-plus-ingress.yaml deployment/nginx-plus-ingress-sm.yaml
  vi deployment/nginx-plus-ingress.yaml

コメントを付与した行を適切な内容に修正してください

.. code-block:: yaml
  :linenos:
  :caption: deployment/nginx-plus-ingress.yaml
  :emphasize-lines: 7-12,17,25,26,30,32-35,40-44

  ** 省略 **
   metadata:
     labels:
       app: nginx-ingress
       nsm.nginx.com/deployment: nginx-ingress
       spiffe.io/spiffeid: "true"
     annotations:
       prometheus.io/scrape: "true"         #
       prometheus.io/port: "9113"           # 
       prometheus.io/scheme: http           # 
       nsm.nginx.com/enable-ingress: "true" #
       nsm.nginx.com/enable-egress: "true"  # 
  ** 省略 **
  spec:
     serviceAccountName: nginx-ingress
     containers:
     - image: registry.example.com/root/nic/nginxplus-ingress-nap-dos:2.1.0  # 対象のレジストリを指定してください
     imagePullPolicy: IfNotPresent
     name: nginx-plus-ingress
  ** 省略 **
     args:
        - -nginx-plus
        - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
        - -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret
        - -enable-app-protect                            # App Protect WAFを有効にします
        - -enable-app-protect-dos                        # App Protect DoSを利用する場合、有効にします
        #- -v=3 # Enables extensive logging. Useful for troubleshooting.
        #- -report-ingress-status
        #- -external-service=nginx-ingress
        - -enable-prometheus-metrics                     #
        #- -global-configuration=$(POD_NAMESPACE)/nginx-configuration
        - -enable-preview-policies                       # OIDCに必要となるArgsを有効にします
        - -enable-snippets                               # OIDCで一部設定を追加するためsnippetsを有効にします
        - -spire-agent-address=/run/spire/sockets/agent.sock  # 
        - -enable-latency-metrics                        # 
        #- -enable-internal-routes
        # Needed for UDP
        # - -enable-preview-policies
        # - -global-configuration=nginx-ingress/nginx-configuration
    volumes:                                             # 
    - hostPath:
        path: /run/spire/sockets
        type: DirectoryOrCreate
      name: spire-agent-socket

修正したマニフェストを指定しPodを作成します。

.. code-block:: cmdin
   
  ## cd ~/kubernetes-ingress/deployments
  kubectl apply -f deployment/nginx-plus-ingress-sm.yaml
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  deployment.apps/nginx-ingress created

.. code-block:: cmdin
   
  kubectl get pods --namespace=nginx-ingress | grep nginx-ingress
   
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nginx-ingress-7f67968b56-d8gf5       1/1     Running   0          3s

.. code-block:: cmdin
   
  kubectl get deployment -n nginx-ingress | grep nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nginx-ingress   1/1     1            1           2m52s

`6. NGINX Ingress Controller を外部へ NodePort で公開する <https://f5j-nginx-ingress-controller-lab1.readthedocs.io/en/latest/class1/module2/module2.html#nginx-ingress-controller-nodeport>`__ に従って操作を行った後、次のNGINX Service Meshのセットアップに進んでください

