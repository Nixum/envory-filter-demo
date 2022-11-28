# envory-filter-demo
envory-filter-demo

tinygo build -o ./main.go.wasm -scheduler=none -target=wasi ./main.go


function envoy_on_request(request_handle)
    request_handle:logInfo("****** start *******")
    request_handle:logInfo("----- enter envoy_on_request : Set x-tenant-name by x-tenant-id")
    local headers = request_handle:headers()
    for key,value in pairs(headers) do
        request_handle:logInfo(key.." "..value)
    end

    request_handle:logInfo("----metadata")
    local metad = request_handle:metadata()
    for keym,valuem in pairs(metad) do
        request_handle:logInfo(keym,valuem)
    end
    request_handle:logInfo("****** finished *******")
end


sidecar.istio.io/userVolume: '[{"name":"wasmfilters-dir","configMap": {"name": "request-printer"}}]'
sidecar.istio.io/userVolumeMount: '[{"mountPath":"/var/local/lib/wasm-filters","name":"wasmfilters-dir"}]'


kubectl logs -l app=o2oms-gateway,version=stable -n gcp -c o2oms-gateway --tail=100 -f

kubectl logs -l app=o2oms-gateway,version=beta -n gcp -c o2oms-gateway --tail=100 -f

---

kubectl logs -l app=merchant,version=stable -n gcp -c merchant --tail=100 -f

kubectl logs -l app=merchant,version=beta -n gcp -c merchant --tail=100 -f

---

kubectl logs -l app=auth,version=stable -n gcp -c auth --tail=100 -f

kubectl logs -l app=auth,version=beta -n gcp -c auth --tail=100 -f

---
kubectl logs -l app=mpms,version=stable -n gcp -c mpms --tail=100 -f

kubectl logs -l app=mpms,version=beta -n gcp -c mpms --tail=100 -f