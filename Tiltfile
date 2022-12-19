version_settings(constraint='>=0.22.2')

docker_compose(
    './compose/docker-compose.yml',
    env_file='.env.development',
    project_name='iotportaldevicemanagement',
)


# Certificates
local_resource(
    'iotportaldevicemanagement-builder',
    'docker build -f build/Dockerfile.production -t iotportaldevicemanagement-builder \
    --build-arg HOSTNAME=192.168.0.161 .',
    deps=['./build'],
)

local_resource(
    'certificates_volumes',
    'docker run -d --name=iotportaldevicemanagement-builder \
    -v iotportaldevicemanagement_certificates:/certificates \
    -v iotportaldevicemanagement_nginx-certificates:/nginx-certificates \
    -v iotportaldevicemanagement_postgres-certificates:/postgres-certificates \
    -v iotportaldevicemanagement_redis-certificates:/redis-certificates \
    -v iotportaldevicemanagement_vernemq-certificates:/vernemq-certificates iotportaldevicemanagement-builder',
    resource_deps=['iotportaldevicemanagement-builder'],
)

local_resource(
    'certificates_volumes_delete_container',
    'docker container stop iotportaldevicemanagement-builder && docker container rm iotportaldevicemanagement-builder',
    resource_deps=['certificates_volumes'],
)

include('../api/Tiltfile')
include('../nginx/Tiltfile')
include('../postgres/Tiltfile')
include('../redis/Tiltfile')
include('../web/Tiltfile')

# VerneMQ
docker_build(
    'iotportaldevicemanagement-vernemq',
    '../docker-vernemq/',
    dockerfile='../docker-vernemq/Dockerfile.alpine',
)


# Docker Compose resources
dc_resource('api', resource_deps=['certificates_volumes',  'certificates_volumes_delete_container'])
dc_resource('nginx', resource_deps=['certificates_volumes',  'certificates_volumes_delete_container'])
dc_resource('postgres', resource_deps=['certificates_volumes',  'certificates_volumes_delete_container'])
dc_resource('queue-worker', resource_deps=['certificates_volumes',  'certificates_volumes_delete_container'])
dc_resource('redis', resource_deps=['certificates_volumes', 'certificates_volumes_delete_container',])
dc_resource('vernemq', resource_deps=['certificates_volumes',  'certificates_volumes_delete_container'])

tiltfile_path = config.main_path

if config.tilt_subcommand == 'up':
    print("""
    \033[32m\033[32mHello World from iot-portal-device-management!\033[0m

    If this is your first time using Tilt and you'd like some guidance, we've got a tutorial to accompany this project:
    https://docs.tilt.dev/tutorial

    If you're feeling particularly adventurous, try opening `{tiltfile}` in an editor and making some changes while Tilt is running.
    What happens if you intentionally introduce a syntax error? Can you fix it?
    """.format(tiltfile=tiltfile_path))
