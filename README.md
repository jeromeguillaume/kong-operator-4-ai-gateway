# Konnect material for Kong Operator and Kong AI Gateway
See below 2 templates for deploying the Kong AI Gateway in Konnect. **There is no difference in terms of plugins capabilities between the two deployments**. Both deployments can be deployed in the same cluster and managed by the same Kong Operator.

The Namespaces are for:
- Kong Operator: `kong-system`
- Hybrid DP: `kong`
- Ingress: `kong-ingress`

## Prerequisite: deploy the Kong Operator and CRDs
- Apply only the `prerequisites` part in the [Kong Operator](https://developer.konghq.com/operator/konnect/crd/control-planes/hybrid/#prerequisites) documentation.
  - Note: do not create the `KonnectAPIAuthConfiguration` because it's done later
- Install the CRDs for [`Gateway API`](https://developer.konghq.com/operator/dataplanes/get-started/kic/install/#install-crds)

## Template for Hybrid DP
It delivers a read/write Control Plane in Konnect. The configuration is based on the following CRD Kinds:
- KonnectGatewayControlPlane
- DataPlane
- KongService (optional)
- KongRoute
- KongPlugin
- KongPluginBinding
- KongConsumer
- All the CRDs are defined [here](https://developer.konghq.com/operator/reference/custom-resources/)

The documentation is here: [Hybrid DP for Konnect](https://developer.konghq.com/operator/dataplanes/get-started/hybrid/install/)

### How to deploy the Hybrid DP
The material is here: [template-hybrid-dp](/template-hybrid-dp)
  - Replace in `0-pat-secret.yaml` the `***TO_BE_REPLACED***` by your base64 token value
  - Replace in `4-kongService-Gemini.yaml`  the `***_TO_BE_REPLACED_***` by your Gemini key value
  - Apply the kubernetes yaml files one by one
  - Get the Public IP address of the DP (`kubectl -n kong get svc`) and export the value in `PROXY_HYBRID_CP`
  - Execute a Gemini prompt:
  ```shell
  curl --request POST \
  --url http://$PROXY_HYBRID_CP/gemini/api/v1 \
  --header 'Content-Type: application/json' \
  --header 'x-llm: gemini4' \
  --data '{
	"messages": [
		{
			"role": "system",
			"content": "You are a mathematician"
		},
		{
			"role": "user",
			"content": "What is 1+1?"
		}
	]
  }'
  ```

## Template for Ingress for API Gateway
It delivers a read only Control Plane in Konnect. Some restrictions appears like the integration to the Developer Portal and the apikey or Dynamic Client Registration that the Developer Portal can provide
- KonnectGatewayControlPlane
- GatewayConfiguration
- GatewayClass
- Gateway
- Service (optional)
- HTTPRoute
- KongClusterPlugin
- KongPlugin
  - The binding of the plugin (for the Service or Route) is done with:
```yaml
annotations:
  konghq.com/plugins: ai-proxy-gemini
```
- KongConsumer
- All the CRDs are defined [here](https://developer.konghq.com/operator/reference/custom-resources/)


The documentation is here: [Ingress for Konnect](https://developer.konghq.com/operator/dataplanes/get-started/kic/create-gateway/#kong-konnect)

### How to deploy the Ingress
The material is here: [template-ingress](/template-ingress)
  - Replace in `0-pat-secret.yaml` the `***TO_BE_REPLACED***` by your base64 token value
  - Replace in `4-kongService-Gemini.yaml`  the `***_TO_BE_REPLACED_***` by your Gemini key value
  - Apply the kubernetes yaml files one by one
  - Get the Public IP address of the DP (`kubectl -n kong-ingress get svc`) and export the value in `PROXY_INGRESS`
  - Execute a Gemini prompt:
  ```shell
  curl --request POST \
  --url http://$PROXY_INGRESS/gemini/api/v1 \
  --header 'Content-Type: application/json' \
  --header 'x-llm: gemini4' \
  --data '{
	"messages": [
		{
			"role": "system",
			"content": "You are a mathematician"
		},
		{
			"role": "user",
			"content": "What is 1+1?"
		}
	]
  }'
  ```

## Konnect - API Gateway managed by the Kong Operator
![Alt text](/images/1-konnect-gw-manager.png?raw=true "Konnect - API Gateway managed by the Kong Operator")