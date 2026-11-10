# Load the restart_process extension
load('ext://restart_process', 'docker_build_with_restart')

### K8s Config

# Uncomment to use secrets
# k8s_yaml('./infra/development/k8s/secrets.yaml')

k8s_yaml('./infra/development/k8s/app-config.yaml')


### API Gateway Config

gateway_compile_cmd = 'CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o build/api-gateway ./services/api-gateway'
if os.name == 'nt':
  gateway_compile_cmd = './infra/development/docker/api-gateway-build.bat'

local_resource(
  'api_gateway-compile',
  gateway_compile_cmd,
  deps=['./services/api-gateway', './shared'], labels="compiles"
)

docker_build_with_restart(
  'Rido/api-gateway',
  '.',
  entrypoint=['/app/build/api-gateway'],
  dockerFile='./infra/development/docker/api-gateway.Dockerfile',
  only=[
    './build/api-gateway',
    './shared'
  ],
  live_update=[
    sync('./build', '/app/build'),
    sync('./shared', '/app/shared')
  ]
)

k8s_yaml('./infra/development/k8s/api-gateway-deployment.yaml')
k8s_resources(
  'api-gateway',
  port_forwards=8081,
  resource_deps=['api-gateway-compile'],
  labels="services"
)

### Trip Service Config


### Web Frontend Config

docker_build(
  'Rido/web',
  '.',
  dockerFile='./infra/development/docker/web.Dockerfile'
)

k8s_yaml('./infra/development/k8s/web-deployment.yaml')
k8s_resources(
  'web',
  port_forwards=3000,
  labels="frontend"
)