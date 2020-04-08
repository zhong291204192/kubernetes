# "datasource": "prometheus"
# 选项框（variables）
    label_values(container_cpu_usage_seconds_total,pod)



# 堆内存监控
	jvm_memory_bytes_used{area="heap",pod=~"$pod"} / 	jvm_memory_bytes_max{pod=~"$pod"}
	{{ pod }}



# POD Memory 使用百分比
	(container_memory_usage_bytes{container_name!="POD",container_name!="",pod_name=~"$pod"} - container_memory_cache{container_name!="POD",container_name!="",pod_name=~"$pod"})/container_spec_memory_limit_bytes{container_name!="POD",container_name!="",pod_name=~"$pod"}
	{{ pod_name }}


# POD CPU 使用百分比
	(sum by (pod)(rate(container_cpu_usage_seconds_total{job="kubelet", image!="",container_name!="POD",pod_name=~"$pod"}[1m])))/(sum by (pod)(kube_pod_container_resource_limits_cpu_cores{pod=~"$pod"}))
	{{ pod }}


# POD 应用GC情况
	increase(jvm_gc_collection_seconds_sum{pod=~"$pod"}[$__interval])
	{{ pod }}--{{gc}}


# java使用线程数
	jvm_threads_current{pod=~"$pod"}
	{{pod}}


# java 死锁线程
	jvm_threads_deadlocked{pod=~"$pod"}
	{{pod}}


# POD Network I/O 入口流量
	sort_desc(sum by (pod_name) (rate(container_network_transmit_bytes_total{job="kubelet",pod_name!="",pod_name=~"$pod"}[1m])))
	{{ pod_name }}

