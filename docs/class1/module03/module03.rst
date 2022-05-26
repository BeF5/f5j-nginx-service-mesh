NSM サンプルアプリケーションのデプロイ
####

サンプルアプリケーションのデプロイ
====

サンプルアプリケーションをデプロイします。

ここではbookinfoというアプリケーションをデプロイし、NSMの基本的な動作を確認します。
予め作成した ``staging`` というNamespaceを指定します。

デプロイするアプリケーションの構成は以下です。

   .. image:: ./media/bookinfo-structure.jpg
      :width: 400

以下コマンドでアプリケーションをデプロイします。

.. code-block:: cmdin

  kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/bookinfo/platform/kube/bookinfo.yaml -n staging

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  service/details created
  serviceaccount/bookinfo-details created
  deployment.apps/details-v1 created
  service/ratings created
  serviceaccount/bookinfo-ratings created
  deployment.apps/ratings-v1 created
  service/reviews created
  serviceaccount/bookinfo-reviews created
  deployment.apps/reviews-v1 created
  deployment.apps/reviews-v2 created
  deployment.apps/reviews-v3 created
  service/productpage created
  serviceaccount/bookinfo-productpage created
  deployment.apps/productpage-v1 created

リソースを確認
====

Podの状態を確認します。

.. code-block:: cmdin

  kubectl get pod -n staging

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                              READY   STATUS            RESTARTS   AGE
  details-v1-7f4669bdd9-djv87       0/2     PodInitializing   0          58s
  productpage-v1-5586c4d4ff-zhljk   0/2     PodInitializing   0          57s
  ratings-v1-6cf6bc7c85-56wnh       0/2     PodInitializing   0          58s
  reviews-v1-7598cc9867-trcwx       0/2     PodInitializing   0          58s
  reviews-v2-6bdd859457-d7r9s       0/2     PodInitializing   0          58s
  reviews-v3-6c98f9d7d7-xmrrt       0/2     PodInitializing   0          58s

上記の結果はPod作成中となりますが、対象のPodが ``0/2`` となっていることに注目してください。これはNSMによりSideCarが挿入される状態であることを示します。
一定時間立つと STATUS が ``Running`` となっていることが確認できます

.. code-block:: cmdin

  kubectl get pod -n staging

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                              READY   STATUS    RESTARTS   AGE
  details-v1-7f4669bdd9-djv87       2/2     Running   0          3m37s
  productpage-v1-5586c4d4ff-zhljk   2/2     Running   0          3m36s
  ratings-v1-6cf6bc7c85-56wnh       2/2     Running   0          3m37s
  reviews-v1-7598cc9867-trcwx       2/2     Running   0          3m37s
  reviews-v2-6bdd859457-d7r9s       2/2     Running   0          3m37s
  reviews-v3-6c98f9d7d7-xmrrt       2/2     Running   0          3m37s

このPodの中から ``details-v1-7f4669bdd9-djv87`` の詳細を確認します。
Pod名は皆様のアウトプットに合わせて変更ください

.. code-block:: cmdin

  ## kubectl describe pod <pod名> -n staging
  kubectl describe pod details-v1-7f4669bdd9-djv87 -n staging

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 20-43,58-85,107-122
  
  Name:         details-v1-7f4669bdd9-djv87
  Namespace:    staging
  Priority:     0
  Node:         ip-10-1-1-9/10.1.1.9
  Start Time:   Wed, 25 May 2022 15:31:25 +0000
  Labels:       app=details
                nsm.nginx.com/deployment=details-v1
                pod-template-hash=7f4669bdd9
                spiffe.io/spiffeid=true
                version=v1
  Annotations:  cni.projectcalico.org/containerID: f369cb16ad3eecff731423d3914893ae5ddc8e60f5e5c14f3ca1048a7858aebf
                cni.projectcalico.org/podIP: 192.168.127.49/32
                cni.projectcalico.org/podIPs: 192.168.127.49/32
                injector.nsm.nginx.com/status: injected
  Status:       Running
  IP:           192.168.127.49
  IPs:
    IP:           192.168.127.49
  Controlled By:  ReplicaSet/details-v1-7f4669bdd9
  Init Containers:
    nginx-mesh-init:
      Container ID:  docker://5a123e11c03716e25d03a451b6ab16dce274c90be68cb3c3318bd979c365a429
      Image:         docker-registry.nginx.com/nsm/nginx-mesh-init:1.4.0
      Image ID:      docker-pullable://docker-registry.nginx.com/nsm/nginx-mesh-init@sha256:7397d2f0ffd572c227907f40e3cb56fb9198d1ba69a7793648f229eeb9000c32
      Port:          <none>
      Host Port:     <none>
      Args:
        --ignore-incoming-ports
        8887
        --outgoing-udp-port
        8908
        --incoming-udp-port
        8909
      State:          Terminated
        Reason:       Completed
        Exit Code:    0
        Started:      Wed, 25 May 2022 15:31:37 +0000
        Finished:     Wed, 25 May 2022 15:31:37 +0000
      Ready:          True
      Restart Count:  0
      Environment:    <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-566p7 (ro)
  Containers:
    details:
      Container ID:   docker://71b07348dff5e73b2165260333a64d6412c91ba6659596dec5f0afefe6e7b164
      Image:          docker.io/istio/examples-bookinfo-details-v1:1.16.4
      Image ID:       docker-pullable://istio/examples-bookinfo-details-v1@sha256:30d373ab66194606eecd0d17809446d61775eafbff1600d2f6f771e7ca777e64
      Port:           9080/TCP
      Host Port:      0/TCP
      State:          Running
        Started:      Wed, 25 May 2022 15:32:58 +0000
      Ready:          True
      Restart Count:  0
      Environment:    <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-566p7 (ro)
    nginx-mesh-sidecar:
      Container ID:  docker://5e339cd960d2123e5f9f875237b3be52b5d21b7e6e8c294d96d62482d342881e
      Image:         docker-registry.nginx.com/nsm/nginx-mesh-sidecar:1.4.0
      Image ID:      docker-pullable://docker-registry.nginx.com/nsm/nginx-mesh-sidecar@sha256:ee3712c909c44dac973ce4efa3dc4b17dee9773b7742b06b5eb6f3ec86fcd516
      Port:          8887/TCP
      Host Port:     0/TCP
      Args:
        -s
        9080
        -n
        details-v1
        --namespace
        nginx-mesh
        -d
        example.org
      State:          Running
        Started:      Wed, 25 May 2022 15:33:49 +0000
      Ready:          True
      Restart Count:  0
      Environment:
        MY_DEPLOY_NAME:      details-v1
        MY_NAMESPACE:        staging (v1:metadata.namespace)
        MY_POD_NAME:         details-v1-7f4669bdd9-djv87 (v1:metadata.name)
        MY_POD_IP:            (v1:status.podIP)
        MY_SERVICE_ACCOUNT:   (v1:spec.serviceAccountName)
      Mounts:
        /run/spire/sockets from spire-agent-socket (ro)
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-566p7 (ro)
  Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
  Volumes:
    kube-api-access-566p7:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      ConfigMapOptional:       <nil>
      DownwardAPI:             true
    spire-agent-socket:
      Type:          HostPath (bare host directory volume)
      Path:          /run/spire/sockets
      HostPathType:  DirectoryOrCreate
  QoS Class:         BestEffort
  Node-Selectors:    <none>
  Tolerations:       node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                     node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type    Reason     Age    From               Message
    ----    ------     ----   ----               -------
    Normal  Scheduled  4m35s  default-scheduler  Successfully assigned staging/details-v1-7f4669bdd9-djv87 to ip-10-1-1-9
    Normal  Pulling    4m29s  kubelet            Pulling image "docker-registry.nginx.com/nsm/nginx-mesh-init:1.4.0"
    Normal  Pulled     4m23s  kubelet            Successfully pulled image "docker-registry.nginx.com/nsm/nginx-mesh-init:1.4.0" in 5.303524573s
    Normal  Created    4m23s  kubelet            Created container nginx-mesh-init
    Normal  Started    4m23s  kubelet            Started container nginx-mesh-init
    Normal  Pulling    4m22s  kubelet            Pulling image "docker.io/istio/examples-bookinfo-details-v1:1.16.4"
    Normal  Pulled     3m4s   kubelet            Successfully pulled image "docker.io/istio/examples-bookinfo-details-v1:1.16.4" in 1m18.504345052s
    Normal  Created    3m3s   kubelet            Created container details
    Normal  Started    3m2s   kubelet            Started container details
    Normal  Pulling    3m2s   kubelet            Pulling image "docker-registry.nginx.com/nsm/nginx-mesh-sidecar:1.4.0"
    Normal  Pulled     2m12s  kubelet            Successfully pulled image "docker-registry.nginx.com/nsm/nginx-mesh-sidecar:1.4.0" in 50.471507038s
    Normal  Created    2m12s  kubelet            Created container nginx-mesh-sidecar
    Normal  Started    2m11s  kubelet            Started container nginx-mesh-sidecar

出力が多くなっていますが、主要な内容を以下に示します。
- 最下部の ``Event`` を見ると ``nginx-mesh-sidecar`` 、 ``nginx-mesh-init`` 、アプリケーションである ``bookinfo-details-v1`` が実行されています。
- ``Container`` の通り、 ``nginx-mesh-sidecar`` 、 ``details`` が実行されています。

それでは、bookinfoに接続するためIngressをデプロイします。

.. code-block:: cmdin

  kubectl apply -f  bookinfo-ingress-staging.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ingress.networking.k8s.io/bookinfo-ingress created

デプロイされたことを確認します。

.. code-block:: cmdin

  kubectl get ingress -A

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAMESPACE    NAME                 CLASS    HOSTS                    ADDRESS   PORTS   AGE
  nginx-mesh   grafana-ingress      nginx2   grafana.example.com                80      47m
  nginx-mesh   jaeger-ingress       nginx2   jaeger.example.com                 80      47m
  nginx-mesh   prometheus-ingress   nginx2   prometheus.example.com             80      48m
  staging      bookinfo-ingress     nginx    bookinfo.example.com               80      4m31s

動作確認
====

Chromeで ``http://bookinfo.example.com/`` へ接続してください

   .. image:: ./media/bookinfo-top.jpg
      :width: 400

下部のリンク ``Normal User`` をクリックしてください。画面を更新すると表示の内容が変わることが確認できます。

   .. image:: ./media/bookinfo-app.jpg
      :width: 400

これらのアプリケーションはNSMがデプロイされております。CLIを使って通信の内容を確認することができます。

.. code-block:: cmdin

  nginx-meshctl top -n staging

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Deployment      Incoming Success  Outgoing Success  NumRequests
  productpage-v1                    100.00%           1
  reviews-v3      100.00%           100.00%           2
  ratings-v1      100.00%                             1

サンプルアプリケーションをデプロイし、NSMを使った通信が行われていることが確認できました。

サービス間のRateLimit
####

設定内容の確認
====

適用する内容は以下の内容です。

.. code-block:: bash
  :linenos:
  :caption: ratelimit1.yaml (~/f5j-nsm-lab/example/配下のファイル)
  :emphasize-lines: 7-17

  apiVersion: specs.smi.nginx.com/v1alpha2
  kind: RateLimit
  metadata:
    name: ratelimit-v1
    namespace: staging
  spec:
    sources:
    - kind: Deployment
      name: productpage-v1
      namespace: staging
    destination:
      kind: Service
      name: reviews
      namespace: staging
    name: 1rm
    rate: 1r/m
    delay: nodelay

- ``source`` が送信元となるサービスを指定しています
- ``destination`` が宛先となるサービスを指定しています
- ``1r/m`` で1分辺りに1リクエストとなるRateLimitを指定しています


動作確認
====

Puttyを右クリックし、 ``Duplicate Session`` をクリックし、ターミナルを追加してください。
新しく追加したターミナルで以下コマンドを実行してください。

.. code-block:: cmdin

  while : ; do sleep 5; curl -sH "Host: bookinfo.example.com" 127.0.0.1/productpage | grep -e "Book Reviews" -e "Sorry," ; done ;

.. code-block:: bash
  :linenos:
  :caption: ターミナル出力結果

      <h4 class="text-center text-primary">Book Reviews</h4>
      <h4 class="text-center text-primary">Book Reviews</h4>
      <h4 class="text-center text-primary">Book Reviews</h4>
    ...

5秒ごとにWebページへアクセスしていることがわかります。

こちらに対しRateLimitのポリシーを適用します。

それではRateLimitを実際に反映します。Webページへアクセスを行っているターミナルとは別のターミナルで、Ratelimitを適用してください。

.. code-block:: cmdin

  cd ~/f5j-nsm-lab/example/
  kubectl apply -f ratelimit1.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ratelimit.specs.smi.nginx.com/ratelimit-v1 created

設定が反映されました。その後、ターミナルの出力を確認すると、
以下のように表示が変更したことが確認できます。

.. code-block:: bash
  :linenos:
  :caption: ターミナル出力結果
  :emphasize-lines: 2,3

      <h4 class="text-center text-primary">Book Reviews</h4>
      <p>Sorry, product reviews are currently unavailable for this book.</p>
      <p>Sorry, product reviews are currently unavailable for this book.</p>
    ...

ブラウザでこの挙動を確認することが可能です。
Chrome で ``http://bookinfo.example.com/productpage`` にアクセスし、更新ボタンを数回押してください

複数回実行すると、以下のようなエラーメッセージが表示されレビューの内容が閲覧できない状態が発生することがわかります。

   .. image:: ./media/bookinfo-ratelimit1.jpg
      :width: 400

RateLimitにより、productpageというアプリケーションが内部で別のサービスにアクセスする通信量を制御出来ることが確認できました。


RateLimitのポリシーを削除します。

.. code-block:: cmdin

  kubectl delete -f ratelimit1.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ratelimit.specs.smi.nginx.com "ratelimit-v1" deleted


ターミナルのループが不要であれば ``Ctrl-C`` で停止してください


条件を指定したRateLimit
####

設定内容の確認
====

先程の操作では、Curl、ブラウザ共にRateLimitのポリシーが適用されていました。
今度は対象となる通信をしていし、 ``Curl`` による接続のみが対象となるよう指定します。

まず、ポリシーの内容を確認します。

.. code-block:: bash
  :linenos:
  :caption: ratelimit2.yaml (~/f5j-nsm-lab/example/配下のファイル)
  :emphasize-lines: 18-22
  
  apiVersion: specs.smi.nginx.com/v1alpha2
  kind: RateLimit
  metadata:
    name: ratelimit-v2
    namespace: staging
  spec:
    sources:
    - kind: Deployment
      name: productpage-v1
      namespace: staging
    destination:
      kind: Service
      name: reviews
      namespace: staging
    name: 1rm
    rate: 1r/m
    delay: nodelay
    rules:
    - kind: HTTPRouteGroup
      name: route-group
      matches:
      - target-ua

- 基本的な内容は先程のポリシーと同様です。末尾に ``rules`` が追加されています
- ``rules`` で kind ``HTTPRouteGroup`` を指定しており、条件の詳細が ``target-ua`` となります

.. code-block:: bash
  :linenos:
  :caption: httproutegroup-ac1.yaml (~/f5j-nsm-lab/example/配下のファイル)
  :emphasize-lines: 7-10

  apiVersion: specs.smi-spec.io/v1alpha3
  kind: HTTPRouteGroup
  metadata:
    name: route-group
    namespace: staging
  spec:
    matches:
    - name: target-ua
      headers:
      - User-Agent: ".*curl.*"

- ``matches`` に対象とする条件を示しています。 HTTP HeaderのUser-Agentに ``curl`` という文字列が含まれる通信を対象とします。

これらのポリシーを適用することにより、CurlがRateLimitの対象となり、その他通信は対象とならない制御になります


動作確認
====

2つ目のターミナルで先程と同様のリクエストを実行します。

.. code-block:: cmdin

  while : ; do sleep 5; curl -sH "Host: bookinfo.example.com" 127.0.0.1/productpage | grep -e "Book Reviews" -e "Sorry," ; done ;

現在はポリシーを適用していないためRateLimitが発生しないことを確認してください。

それではポリシーを適用します。

.. code-block:: cmdin

  kubectl apply -f httproutegroup-ac1.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  httproutegroup.specs.smi-spec.io/route-group created

.. code-block:: cmdin

  kubectl apply -f ratelimit2.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ratelimit.specs.smi.nginx.com/ratelimit-v2 created

``Curl`` コマンドでアクセスしているターミナルでは一定時間経過後、先程と同様にエラーが表示されていることが確認できます。

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2,3

      <h4 class="text-center text-primary">Book Reviews</h4>
      <p>Sorry, product reviews are currently unavailable for this book.</p>
      <p>Sorry, product reviews are currently unavailable for this book.</p>
    ...

先程と同様に通信が制限されていることが確認できます。

次にブラウザでアクセスします。ブラウザでアクセスした際には先程のように制限はされず、正しく閲覧出来ることが確認できます。

この様に条件を指定することで、対象の通信を識別し制限の対象とする通信を限定することが可能です

NSMによる通信ステータスの確認
####

ブラウザで Jaeger にアクセスし、更新ボタンを教えてください
いくつかの通信が発生したことにより、対象となるサービスが複数に増えていることが確認できます
- Jaeger: ``http://jaeger.example.com:8080/``

   .. image:: ./media/jaeger-ratelimit2.jpg
      :width: 400

サービスを指定し、 ``Find Traces`` をクリックすることで詳細を確認することが可能です

Grafanaではいくつかのステータスを見ることができます
- Grafana: ``http://grafana.example.com:8080/``

   .. image:: ./media/grafana-ratelimit2.jpg
      :width: 400

Prometheusはステータスを取得しています
Prometheusでは特定のステータすの詳細を確認することが可能です。
- Prometheus: ``http://prometheus.example.com:8080/``

   .. image:: ./media/prometheus-ratelimit2_1.jpg
      :width: 400

例えば、 ``nginxplus_http_requests_total`` を指定し、 ``Execute`` をクリックすると、Prometheusが観測した http_request の数が確認できます。
Graphのタブをクリックするとどの様に値が変化しているか確認することが可能です。

   .. image:: ./media/prometheus-ratelimit2_2.jpg
      :width: 400
