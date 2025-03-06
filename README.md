# Operator SDK Tutorial

## 1. Installation & Preseqs

Operator SDK'yi yüklemek için orijinal dökümantasyonu takip edebilirsiniz: [Operator SDK Installation Guide](https://sdk.operatorframework.io/docs/installation/)

-  go
-  make
-  docker
-  kubectl işlemlerini yapabilen ve cluster-admin yetkilerine sahip kullanıcı

## 2. Initialize the Operator

Öncelikle proje klasörünü oluşturup içine girin:

```sh
mkdir  memcached-op
cd  memcached-op
```

Ardından operator projenizi başlatmak için aşağıdaki komutu çalıştırın:

```sh
operator-sdk  init  --domain=example.com  --repo=github.com/example/memcached-operator
```

-  `--domain=example.com`: Operatörünüzün CRD'leri (Custom Resource Definitions) için kullanılacak domain adını belirtir. Genellikle organizasyonunuzun domain adı tercih edilir.

-  `--repo=example-repo.com/projects/memcached-operator`: Operator projesinin bulunduğu Go reposunun adresini tanımlar. Bu, modüllerin import edilme yolunu belirler. (Go modülü başlatmak için `go mod init example-repo.com/projects/memcached-operator` komutuna benzetilebilir.)

## 3. API Creation

Operator SDK varsayılan olarak **Single Group API** şeklinde çalışır. Ancak, **Multi Group API** desteğini açmak isterseniz proje dizininde aşağıdaki komutu kullanabilirsiniz:

```sh
operator-sdk  edit  --multigroup=true
```

Eğer mevcut bir operatörunuz varsa ve bunu **multi group** mimariye dönüştürmek istiyorsanız, aşağıdaki dökümantasyonu takip edebilirsiniz:

📖 [Multi-Group Migration Guide](https://book.kubebuilder.io/migration/multi-group)

### Yeni Bir API Oluşturma

Yeni bir API oluşturmak için aşağıdaki komutu çalıştırabilirsiniz:

```sh
operator-sdk  create  api  --group  cache  --version  v1alpha1  --kind  Memcached  --resource  --controller
```

-  `--group cache`: API grubunu belirler.
-  `--version v1alpha1`: API'nin versiyonunu belirtir.
-  `--kind Memcached`: API için oluşturulacak Custom Resource (CR) türünü belirler. **Bu değer büyük harfle başlamalıdır.**
-  `--resource` (Opsiyonel): API için bir **Custom Resource Definition (CRD)** oluşturur.
-  `--controller` (Opsiyonel): API için bir **Controller** oluşturur.

## 4. Building Operator

### 4.1. Single Group API Mimari

Bu aşamada operator calışma mantığını sizlerin kod üzerinden incelemesi amacıyla aşağıdaki işlevlere sahip bir memcached operatoru yazacağız:

-   Memcached Deployment varlığını kontrol etmek, yok ise yaratmak
-   Memcached CR size spec ile Deployment büyüklüğünün ayni olmasını sağlamak

Bu aşamada [Yeni Bir API Oluşturma](#yeni-bir-api-oluşturma) kısmındakı adımı yapmanız gerekiyor.

Basit proje yapısına goz atmak için: [Proje Yapisi](https://book.kubebuilder.io/cronjob-tutorial/basic-project.html)

Projemizin giriş kapısı olan main yapısını gormek için: [main.go](https://book.kubebuilder.io/cronjob-tutorial/empty-main.html)

memcached-op/cmd/main.go içerisinden operatorümüzün spesifik namespace veya namespace setleri üzerinden calışmasını sağlayabilmemiz mümkün. Default olarak operator-sdk bütün namespace'leri izler bu durumu değiştirmek isterseniz. [Operator-scope](https://sdk.operatorframework.io/docs/building-operators/golang/operator-scope/) dokumanina goz atabilirsiniz.

#### 4.1.1. Arranging Types

Bu aşamada artık herhangi bir IDE yardımı ile memcached-op/api/v1alpha1/memcached_types.go dosyasını açıyoruz.

Bu dosya bize sonrasında otomatik olarak verilecek olan CRD'nin tanımlanabilmesi icin duzenlediğimiz dosya. Buraya sahip olmasını istediğimiz özellikleri MemcachedSpec MemcachedStatus ve Memcached struct'ina tanımlamamız gerekiyor. Asağıda bu dosyanın bir örneğini görebilirsiniz.

<details>

<summary>**Düzenlenmiş memcached-op/api/v1alpha1/memcached_types.go dosyası**</summary>

```go 
/*

Copyright 2025.

  

Licensed under the Apache License, Version 2.0 (the "License");

you may not use this file except in compliance with the License.

You may obtain a copy of the License at

  

http://www.apache.org/licenses/LICENSE-2.0

  

Unless required by applicable law or agreed to in writing, software

distributed under the License is distributed on an "AS IS" BASIS,

WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

See the License for the specific language governing permissions and

limitations under the License.

*/

  

package v1alpha1

  

import (

metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"

)

  

// EDIT THIS FILE! THIS IS SCAFFOLDING FOR YOU TO OWN!

// NOTE: json tags are required. Any new fields you add must have json tags for the fields to be serialized.

  

// MemcachedSpec defines the desired state of Memcached

type MemcachedSpec struct {

// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster

// Important: Run "make" to regenerate code after modifying this file

  

// The following markers will use OpenAPI v3 schema to validate the value

// More info: https://book.kubebuilder.io/reference/markers/crd-validation.html

// +kubebuilder:validation:Minimum=1

// +kubebuilder:validation:Maximum=5

// +kubebuilder:validation:ExclusiveMaximum=false

  

// Size defines the number of Memcached instances

// +operator-sdk:csv:customresourcedefinitions:type=spec

Size int32 `json:"size,omitempty"`

  

// Port defines the port that will be used to init the container with the image

// +operator-sdk:csv:customresourcedefinitions:type=spec

ContainerPort int32 `json:"containerPort,omitempty"`

}

  

// MemcachedStatus defines the observed state of Memcached

type MemcachedStatus struct {

// Represents the observations of a Memcached's current state.

// Memcached.status.conditions.type are: "Available", "Progressing", and "Degraded"

// Memcached.status.conditions.status are one of True, False, Unknown.

// Memcached.status.conditions.reason the value should be a CamelCase string and producers of specific

// condition types may define expected values and meanings for this field, and whether the values

// are considered a guaranteed API.

// Memcached.status.conditions.Message is a human readable message indicating details about the transition.

// For further information see: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#typical-status-properties

  

// Conditions store the status conditions of the Memcached instances

// +operator-sdk:csv:customresourcedefinitions:type=status

Conditions []metav1.Condition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,1,rep,name=conditions"`

}

  

// Memcached is the Schema for the memcacheds API

// +kubebuilder:object:root=true

// +kubebuilder:subresource:status

type Memcached struct {

metav1.TypeMeta `json:",inline"`

metav1.ObjectMeta `json:"metadata,omitempty"`

  

Spec MemcachedSpec `json:"spec,omitempty"`

Status MemcachedStatus `json:"status,omitempty"`

}

  

// +kubebuilder:object:root=true

  

// MemcachedList contains a list of Memcached

type MemcachedList struct {

metav1.TypeMeta `json:",inline"`

metav1.ListMeta `json:"metadata,omitempty"`

Items []Memcached `json:"items"`

}

  

func init() {

SchemeBuilder.Register(&Memcached{}, &MemcachedList{})

}

```

</details>

Dosyada gerekli düzenlemeler yapıldıktan sonra;
 
`make generate`

Yukarıdaki makefile hedefi, API'imizin Go türü tanımlarının tüm Kind'larin uygulaması gereken runtime.Object arayüzünü uyguladığından emin olmak için api/v1alpha1/zz_geneated.deepcopy.go dosyasını güncellemek üzere controller-gen yardımcı programını çağıracaktır.

Ardından;

`make manifests` komutu ile CRD'larimizi ve ornek CR'larimizi; memcached-op/config/crd ve memcached-op/config/samples kisminda bulabilirsiniz.

memcached-op/config/samples dizininde bulunan ornek CR'larimizi önceki aşamada güncellediğimiz memcached_types.go dosyasi doğrultusunda spec kısımlarını güncelliyoruz. Aşağıdaki şekilde spec kısmını güncelleyebilirsiniz.

```yaml
spec:
  size: 2
```

#### 4.1.2. Arranging Controller

API için gerekli tanımlarımızı yaptık elimizde artık yönetilmesini istedigimiz bir resource şablonu oluşturduk. Bu aşamada artık bu resourcenin nasıl yönetileceğini tanımlama vakti.

**memcached-op/internal/controller/memcached_controller.go** dosyasını açıyoruz.

İzlememiz gereken adımlar:
- SetupWithManager fonksiyonunun halledilmesi
- Reconcile fonksiyonu araciligiyla reconcile loop logic'inin entegre edilmesi
- CR silindiğinde ilişkili podlarin silinmesinde ihtiyaç olacağı için owner referance eklenmesi
- [Markerlar](https://book.kubebuilder.io/reference/markers) yardımı ile gerekli konfigrasyon ayarlarının yapılması (RBAC vb.)


<details>

<summary>**Düzenlenmiş memcached-op/internal/controller/memcached_controller.go dosyası**</summary>

```go 
/*
Copyright 2025 The Kubernetes authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

package controller

import (
	"context"
	"fmt"

	"time"

	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	apierrors "k8s.io/apimachinery/pkg/api/errors"
	"k8s.io/apimachinery/pkg/api/meta"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/types"
	"k8s.io/utils/ptr"

	"k8s.io/apimachinery/pkg/runtime"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/log"

	cachev1alpha1 "github.com/example/memcached-operator/api/v1alpha1"
)

// Definitions to manage status conditions
const (
	// typeAvailableMemcached represents the status of the Deployment reconciliation
	typeAvailableMemcached = "Available"
)

// MemcachedReconciler reconciles a Memcached object
type MemcachedReconciler struct {
	client.Client
	Scheme *runtime.Scheme
}

// +kubebuilder:rbac:groups=cache.example.com,resources=memcacheds,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=cache.example.com,resources=memcacheds/status,verbs=get;update;patch
// +kubebuilder:rbac:groups=cache.example.com,resources=memcacheds/finalizers,verbs=update
// +kubebuilder:rbac:groups=core,resources=events,verbs=create;patch
// +kubebuilder:rbac:groups=apps,resources=deployments,verbs=get;list;watch;create;update;patch;delete
// +kubebuilder:rbac:groups=core,resources=pods,verbs=get;list;watch

// Reconcile is part of the main kubernetes reconciliation loop which aims to
// move the current state of the cluster closer to the desired state.
// It is essential for the controller's reconciliation loop to be idempotent. By following the Operator
// pattern you will create Controllers which provide a reconcile function
// responsible for synchronizing resources until the desired state is reached on the cluster.
// Breaking this recommendation goes against the design principles of controller-runtime.
// and may lead to unforeseen consequences such as resources becoming stuck and requiring manual intervention.
// For further info:
// - About Operator Pattern: https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
// - About Controllers: https://kubernetes.io/docs/concepts/architecture/controller/
//
// For more details, check Reconcile and its Result here:
// - https://pkg.go.dev/sigs.k8s.io/controller-runtime@v0.20.2/pkg/reconcile
func (r *MemcachedReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	log := log.FromContext(ctx)

	// Fetch the Memcached instance
	// The purpose is check if the Custom Resource for the Kind Memcached
	// is applied on the cluster if not we return nil to stop the reconciliation
	memcached := &cachev1alpha1.Memcached{}
	err := r.Get(ctx, req.NamespacedName, memcached)
	if err != nil {
		if apierrors.IsNotFound(err) {
			// If the custom resource is not found then it usually means that it was deleted or not created
			// In this way, we will stop the reconciliation
			log.Info("memcached resource not found. Ignoring since object must be deleted")
			return ctrl.Result{}, nil
		}
		// Error reading the object - requeue the request.
		log.Error(err, "Failed to get memcached")
		return ctrl.Result{}, err
	}

	// Let's just set the status as Unknown when no status is available
	if len(memcached.Status.Conditions) == 0 {
		meta.SetStatusCondition(&memcached.Status.Conditions, metav1.Condition{Type: typeAvailableMemcached, Status: metav1.ConditionUnknown, Reason: "Reconciling", Message: "Starting reconciliation"})
		if err = r.Status().Update(ctx, memcached); err != nil {
			log.Error(err, "Failed to update Memcached status")
			return ctrl.Result{}, err
		}

		// Let's re-fetch the memcached Custom Resource after updating the status
		// so that we have the latest state of the resource on the cluster and we will avoid
		// raising the error "the object has been modified, please apply
		// your changes to the latest version and try again" which would re-trigger the reconciliation
		// if we try to update it again in the following operations
		if err := r.Get(ctx, req.NamespacedName, memcached); err != nil {
			log.Error(err, "Failed to re-fetch memcached")
			return ctrl.Result{}, err
		}
	}

	// Check if the deployment already exists, if not create a new one
	found := &appsv1.Deployment{}
	err = r.Get(ctx, types.NamespacedName{Name: memcached.Name, Namespace: memcached.Namespace}, found)
	if err != nil && apierrors.IsNotFound(err) {
		// Define a new deployment
		dep, err := r.deploymentForMemcached(memcached)
		if err != nil {
			log.Error(err, "Failed to define new Deployment resource for Memcached")

			// The following implementation will update the status
			meta.SetStatusCondition(&memcached.Status.Conditions, metav1.Condition{Type: typeAvailableMemcached,
				Status: metav1.ConditionFalse, Reason: "Reconciling",
				Message: fmt.Sprintf("Failed to create Deployment for the custom resource (%s): (%s)", memcached.Name, err)})

			if err := r.Status().Update(ctx, memcached); err != nil {
				log.Error(err, "Failed to update Memcached status")
				return ctrl.Result{}, err
			}

			return ctrl.Result{}, err
		}

		log.Info("Creating a new Deployment",
			"Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
		if err = r.Create(ctx, dep); err != nil {
			log.Error(err, "Failed to create new Deployment",
				"Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
			return ctrl.Result{}, err
		}

		// Deployment created successfully
		// We will requeue the reconciliation so that we can ensure the state
		// and move forward for the next operations
		return ctrl.Result{RequeueAfter: time.Minute}, nil
	} else if err != nil {
		log.Error(err, "Failed to get Deployment")
		// Let's return the error for the reconciliation be re-trigged again
		return ctrl.Result{}, err
	}

	// The CRD API defines that the Memcached type have a MemcachedSpec.Size field
	// to set the quantity of Deployment instances to the desired state on the cluster.
	// Therefore, the following code will ensure the Deployment size is the same as defined
	// via the Size spec of the Custom Resource which we are reconciling.
	size := memcached.Spec.Size
	if *found.Spec.Replicas != size {
		found.Spec.Replicas = &size
		if err = r.Update(ctx, found); err != nil {
			log.Error(err, "Failed to update Deployment",
				"Deployment.Namespace", found.Namespace, "Deployment.Name", found.Name)

			// Re-fetch the memcached Custom Resource before updating the status
			// so that we have the latest state of the resource on the cluster and we will avoid
			// raising the error "the object has been modified, please apply
			// your changes to the latest version and try again" which would re-trigger the reconciliation
			if err := r.Get(ctx, req.NamespacedName, memcached); err != nil {
				log.Error(err, "Failed to re-fetch memcached")
				return ctrl.Result{}, err
			}

			// The following implementation will update the status
			meta.SetStatusCondition(&memcached.Status.Conditions, metav1.Condition{Type: typeAvailableMemcached,
				Status: metav1.ConditionFalse, Reason: "Resizing",
				Message: fmt.Sprintf("Failed to update the size for the custom resource (%s): (%s)", memcached.Name, err)})

			if err := r.Status().Update(ctx, memcached); err != nil {
				log.Error(err, "Failed to update Memcached status")
				return ctrl.Result{}, err
			}

			return ctrl.Result{}, err
		}

		// Now, that we update the size we want to requeue the reconciliation
		// so that we can ensure that we have the latest state of the resource before
		// update. Also, it will help ensure the desired state on the cluster
		return ctrl.Result{Requeue: true}, nil
	}

	// The following implementation will update the status
	meta.SetStatusCondition(&memcached.Status.Conditions, metav1.Condition{Type: typeAvailableMemcached,
		Status: metav1.ConditionTrue, Reason: "Reconciling",
		Message: fmt.Sprintf("Deployment for custom resource (%s) with %d replicas created successfully", memcached.Name, size)})

	if err := r.Status().Update(ctx, memcached); err != nil {
		log.Error(err, "Failed to update Memcached status")
		return ctrl.Result{}, err
	}

	return ctrl.Result{}, nil
}

// SetupWithManager sets up the controller with the Manager.
// The Deployment is also watched to ensure its
// desired state in the cluster.
func (r *MemcachedReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		// Watch the Memcached Custom Resource and trigger reconciliation whenever it
		//is created, updated, or deleted
		For(&cachev1alpha1.Memcached{}).
		// Watch the Deployment managed by the Memcached controller. If any changes occur to the Deployment
		// owned and managed by this controller, it will trigger reconciliation, ensuring that the cluster
		// state aligns with the desired state.
		Owns(&appsv1.Deployment{}).
		Complete(r)
}

// deploymentForMemcached returns a Memcached Deployment object
func (r *MemcachedReconciler) deploymentForMemcached(
	memcached *cachev1alpha1.Memcached) (*appsv1.Deployment, error) {
	replicas := memcached.Spec.Size
	image := "memcached:1.6.26-alpine3.19"

	dep := &appsv1.Deployment{
		ObjectMeta: metav1.ObjectMeta{
			Name:      memcached.Name,
			Namespace: memcached.Namespace,
		},
		Spec: appsv1.DeploymentSpec{
			Replicas: &replicas,
			Selector: &metav1.LabelSelector{
				MatchLabels: map[string]string{"app.kubernetes.io/name": "project"},
			},
			Template: corev1.PodTemplateSpec{
				ObjectMeta: metav1.ObjectMeta{
					Labels: map[string]string{"app.kubernetes.io/name": "project"},
				},
				Spec: corev1.PodSpec{
					SecurityContext: &corev1.PodSecurityContext{
						RunAsNonRoot: ptr.To(true),
						SeccompProfile: &corev1.SeccompProfile{
							Type: corev1.SeccompProfileTypeRuntimeDefault,
						},
					},
					Containers: []corev1.Container{{
						Image:           image,
						Name:            "memcached",
						ImagePullPolicy: corev1.PullIfNotPresent,
						// Ensure restrictive context for the container
						// More info: https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted
						SecurityContext: &corev1.SecurityContext{
							RunAsNonRoot:             ptr.To(true),
							RunAsUser:                ptr.To(int64(1001)),
							AllowPrivilegeEscalation: ptr.To(false),
							Capabilities: &corev1.Capabilities{
								Drop: []corev1.Capability{
									"ALL",
								},
							},
						},
						Ports: []corev1.ContainerPort{{
							ContainerPort: 11211,
							Name:          "memcached",
						}},
						Command: []string{"memcached", "--memory-limit=64", "-o", "modern", "-v"},
					}},
				},
			},
		},
	}

	// Set the ownerRef for the Deployment
	// More info: https://kubernetes.io/docs/concepts/overview/working-with-objects/owners-dependents/
	if err := ctrl.SetControllerReference(memcached, dep, r.Scheme); err != nil {
		return nil, err
	}
	return dep, nil
}

```

</details>

Örnek dosyamızı kendi proje dizininizdeki dosya ile değiştirdikten sonra `make manifests` calıştırınız bu komut memcached-op/config/samples/rbac gibi dizinlere kodda yazılan marker' ları işleyecektir.

### 4.2. Multi Group API Mimari Farklari

```sh
operator-sdk  edit  --multigroup=true
```

Bu bölümde bazı ufak farklılıklardan bahsetmemiz gerekiyor. Multi Group API Mimarisini tercih ettiğinizde proje yapısını bir miktar değişiyor. Single Group yapida type'lar api/<version>/ altında bulunurken; Multi Group yapıda api/<group>/<version>/ altında bulunuyor.

Controller için ise durum şu şekilde: Single Group yapıda controller'lar internal/controller/ altında bulunurken; Multi Group yapıda internal/controller/<group>/ altında bulunuyor.

main dosyası bu durumu otamatik olarak handlelayabiliyor.

Single Group API kısmında yaptığımız adımları dizinlere dikkat ederek farklı CRD' ler ile takip ettiğiniz taktirde. Bu yapıda bir operatörü rahatlıkla kurabilirsiniz.

## 5. Build & Deploy

[Gereklilikleri](#1-installation--preseqs) kontrol ediniz.

Projeyi build etmek için (Bu aşamada sudo yetkisi gerekebilir.):

```sh
make docker-build docker-push IMG=<registry>/<project>
```

Projeyi deploy edebilmek için:

```sh
make deploy IMG=<registry>/<project>
```

> **_NOTE:_**  `make install` komutunu sadece CRD'inizi cluster'a yüklemek için kullanabilirsiniz.

CR'larin apply edilmesi:

```sh
kubectl apply -f memcached-op/config/samples/cache_v1alpha1_memcached.yaml
```

Bu aşamada memcached-sample CR'inizda belirlenen size spec'iniz kadar pod' un ayağa kaltığını ve deployment' ının yapıldığını göreceksiniz.

Spec'in degiştirilmesi suretiyle, oluşturduğumuz operatörün pod sayısını kontrolünü test etmek için:

```sh
kuubectl patch memcached memcached-sample -p '{"spec":{"size": 4}}' --type=merge
```

```sh
kubectl get pods
```

## 6. Uninstallation - Undeploy

İlk olarak yaratılan memcached CR'lari silinir ardından operatör undeploy edilir.

```sh
kubectl delete memcached memcached-sample
```

```sh
make undeploy
```