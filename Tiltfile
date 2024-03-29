# Load required tilt extensions

load('ext://helm_resource', 'helm_resource')
load('ext://uibutton', 'cmd_button', 'location')

# ----- Load Configuration from config.yaml -----

configFile = read_yaml('./config.yaml')

# ----- Global Variables -----

# Set to True to enable clustering
# TODO add support for DBStore for DocLib and Elasticsearch
config.define_string("replicas")

cfg = config.parse()
liferay_replicas = cfg.get(
	"replicas", configFile.get('dxp').get('replicas'))

# ----- MySQL -----

helm_resource(
	name='mysql',
	chart='oci://registry-1.docker.io/bitnamicharts/mysql',
	flags=[
		'-f',
		'%s/charts/mysql/values.yaml' % os.getcwd(),
	],
	labels='mysql',
)

# ----- DXP -----

docker_build(
	ref='liferay-tilt/dxp',
	context='./dxp-docker-root',
	dockerfile_contents='''
FROM %s

COPY --chown=liferay:liferay deploy /opt/liferay/deploy
COPY --chown=liferay:liferay osgi /opt/liferay/osgi
COPY --chown=liferay:liferay portal-ext.properties /opt/liferay/portal-ext.properties
COPY --chown=liferay:liferay shielded-container-lib /opt/liferay/tomcat/webapps/ROOT/WEB-INF/shielded-container-lib
COPY --chown=liferay:liferay unicast.xml /opt/liferay/unicast.xml
''' % (configFile.get('dxp').get('image')),
)

helm_resource(
	name='dxp',
	chart='./charts/dxp',
	deps=[
		'./charts/dxp',
	],
	flags=[
		'--set=image=liferay-tilt/dxp',
		'--set=replicas=%s' % liferay_replicas,
	],
	image_deps=['liferay-tilt/dxp'],
	image_keys=['image'],
	resource_deps=[
		'mysql',
	],
	labels='dxp',
)

# ----- Test Resources -----

helm_resource(
	auto_init=False,
	name='test-resources',
	chart='./test-resources',
	deps=[
		'./test-resources',
	],
	resource_deps=[
		'dxp',
	],
	labels='test-resources',
)

# ----- Nuke Data -----

local_resource(
	name='drop-database',
	cmd='./drop-database.sh',
	cmd_bat='./drop-database.cmd',
	trigger_mode=TRIGGER_MODE_MANUAL,
	auto_init=False,
	resource_deps=[
		'mysql',
		'dxp',
	],
	labels='z-drop-database'
)
