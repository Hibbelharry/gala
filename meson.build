project('gala',
	'c', 'vala',
	version: '3.2.0',
	meson_version: '>= 0.46.0',
	license: 'GPL3',
)

is_release = false

gnome = import('gnome')
i18n = import('i18n')
cc = meson.get_compiler('c')

gala_version = meson.project_version()

vala_version_required = '0.28.0'
vala = meson.get_compiler('vala')
if not vala.version().version_compare('>= @0@'.format(vala_version_required))
	error('Valac >= @0@ required!'.format(vala_version_required))
endif


data_dir = join_paths(get_option('prefix'), get_option('datadir'))
plugins_dir = join_paths(get_option('prefix'), get_option('libdir'), meson.project_name(), 'plugins')
pkgdata_dir = join_paths(get_option('prefix'), get_option('datadir'), meson.project_name())
pkglib_dir = join_paths(get_option('prefix'), get_option('libdir'), meson.project_name())

config_inc_dir = include_directories('.')

conf = configuration_data()
conf.set_quoted('PACKAGE', meson.project_name())
conf.set_quoted('PACKAGE_BUGREPORT', 'https://github.com/elementary/gala/issues')
conf.set_quoted('PACKAGE_NAME', meson.project_name())
conf.set_quoted('PACKAGE_STRING', 'Gala')
conf.set_quoted('PACKAGE_TARNAME', meson.project_name())
conf.set_quoted('PACKAGE_URL', 'https://github.com/elementary/gala')
conf.set_quoted('PACKAGE_VERSION', gala_version)
conf.set_quoted('DATADIR', data_dir)
conf.set_quoted('PKGDATADIR', pkgdata_dir)
conf.set_quoted('PKGLIBDIR', pkglib_dir)
conf.set_quoted('PLUGINSDIR', plugins_dir)
conf.set_quoted('RELEASE_NAME', 'Window Manager.')
conf.set_quoted('VERSION', gala_version)
conf.set_quoted('VERSION_INFO', (is_release ? 'Release' : 'Development'))
conf.set_quoted('BUILD_VERSION', gala_version)
configure_file(input: 'config.h.meson', output: 'config.h', configuration: conf)

customconf = configuration_data()
customconf.set('GETTEXT_PACKAGE', meson.project_name())

pkgconf = configuration_data()
pkgconf.set('PACKAGE_VERSION', gala_version)
pkgconf.set('MUTTER_CFLAGS', '')
pkgconf.set('prefix', get_option('prefix'))
pkgconf.set('exec_prefix', '${prefix}')
pkgconf.set('libdir', '${prefix}/@0@'.format(get_option('libdir')))
pkgconf.set('bindir', '${exec_prefix}/@0@'.format(get_option('bindir')))
pkgconf.set('datarootdir', '${prefix}/@0@'.format(get_option('datadir')))
pkgconf.set('datadir', '${datarootdir}')
pkgconf.set('includedir', '${prefix}/@0@'.format(get_option('includedir')))


add_project_arguments([
		'--target-glib=2.44',
		'--hide-internal',
		'--enable-deprecated',
		'--vapidir=' + join_paths(meson.source_root(), 'vapi'),
		'--pkg', 'config',
		'--pkg', 'cogl-fixes',
		'--pkg', 'compat',
	],
	language: 'vala',
)
add_project_arguments([
		'-Wno-discarded-qualifiers',
		'-Wno-incompatible-pointer-types',
		'-Wno-unused',
		'-Wl,--enable-new-dtags',

		'-DGETTEXT_PACKAGE="@0@"'.format(meson.project_name()),
		'-DGNOME_DESKTOP_USE_UNSTABLE_API',

		'-DDATADIR="@0@"'.format(data_dir),
		'-DPKGDATADIR="@0@"'.format(pkgdata_dir),
		'-DPKGLIBDIR="@0@"'.format(pkglib_dir),
		'-DPLUGINDIR="@0@"'.format(plugins_dir),
		'-DSCHEMA="org.pantheon.desktop.gala"',
		'-DRESOURCEPATH="/org/pantheon/desktop/gala"',

		'-DGLIB_VERSION_MIN_REQUIRED=GLIB_VERSION_2_44',
		'-DGLIB_VERSION_MAX_ALLOWED=GLIB_VERSION_2_44',
		'-DGDK_VERSION_MIN_REQUIRED=GDK_VERSION_3_10',
		'-DGDK_VERSION_MAX_ALLOWED=GDK_VERSION_3_10',
	],
	language: 'c',
)


# Dependencies
glib_version_required = '2.44.0'
gtk_version_required = '3.10.0'
plank_version_required = '0.11.0'
gsd_version_required = '3.15.2'

glib_dep = [dependency('gio-2.0', version: '>= @0@'.format(glib_version_required)), dependency('gio-unix-2.0'), dependency('gmodule-2.0')]
gtk_dep = [dependency('gtk+-3.0', version: '>= @0@'.format(gtk_version_required)), dependency('gdk-x11-3.0')]
gee_dep = dependency('gee-0.8')
bamf_dep = dependency('libbamf3')
plank_dep = dependency('plank', version: '>= @0@'.format(plank_version_required))
granite_dep = dependency('granite')
gnome_desktop_dep = dependency('gnome-desktop-3.0')
gsd_dep = dependency('gnome-settings-daemon', version: '>= @0@'.format(gsd_version_required))
m_dep = cc.find_library('m', required: false)

mutter_dep = []
libmutter_dep = []

vala_flags = []

mutter328_dep = dependency('libmutter-2', version: ['>= 3.27', '< 3.29'], required: false)
if mutter328_dep.found()
	mutter_packages_command = [
		'--pkg', 'libmutter-2',
		'--pkg', 'mutter-cogl-2',
		'--pkg', 'mutter-cogl-pango-2',
		'--pkg', 'mutter-cogl-path-2',
		'--pkg', 'mutter-clutter-2'
	]
	libmutter_dep = dependency('libmutter-2', version: '>= 3.27.92')
	pkgconf.set('MUTTER_PKGS', 'libmutter-2 >= 3.27.92')
	mutter_dep = [
		libmutter_dep,
		dependency('mutter-cogl-2'), dependency('mutter-cogl-pango-2'),
		dependency('mutter-cogl-path-2'), dependency('mutter-clutter-2')
	]
	vala_flags += []
	add_project_arguments(['-DCLUTTER_ENABLE_COMPOSITOR_API', '-DCLUTTER_ENABLE_EXPERIMENTAL_API',
		'-DCOGL_ENABLE_EXPERIMENTAL_API', '-DCOGL_ENABLE_EXPERIMENTAL_2_0_API'], language: 'c')
endif

mutter330_dep = dependency('libmutter-3', version: ['>= 3.29.4', '< 3.31'], required: false)
if mutter330_dep.found()
	mutter_packages_command = [
		'--pkg', 'libmutter-3',
		'--pkg', 'mutter-cogl-3',
		'--pkg', 'mutter-cogl-pango-3',
		'--pkg', 'mutter-cogl-path-3',
		'--pkg', 'mutter-clutter-3'
	]
	libmutter_dep = dependency('libmutter-3', version: '>= 3.29.4')
	pkgconf.set('MUTTER_PKGS', 'libmutter-3 >= 3.29.4')
	mutter_dep = [
		libmutter_dep,
		dependency('mutter-cogl-3'), dependency('mutter-cogl-pango-3'),
		dependency('mutter-cogl-path-3'), dependency('mutter-clutter-3')
	]
	vala_flags += ['--define', 'HAS_MUTTER330']
	add_project_arguments(['-DCLUTTER_ENABLE_COMPOSITOR_API', '-DCLUTTER_ENABLE_EXPERIMENTAL_API',
		'-DCOGL_ENABLE_EXPERIMENTAL_API', '-DCOGL_ENABLE_EXPERIMENTAL_2_0_API'], language: 'c')
endif

mutter332_dep = dependency('libmutter-4', version: ['>= 3.31.2', '< 3.34'], required: false)
if mutter332_dep.found()
	mutter_packages_command = [
		'--pkg', 'libmutter-4',
		'--pkg', 'mutter-cogl-4',
		'--pkg', 'mutter-cogl-pango-4',
		'--pkg', 'mutter-cogl-path-4',
		'--pkg', 'mutter-clutter-4'
	]
	libmutter_dep = dependency('libmutter-4', version: '>= 3.31.2')
	pkgconf.set('MUTTER_PKGS', 'libmutter-4 >= 3.31.2')
	mutter_dep = [
		libmutter_dep,
		dependency('mutter-cogl-4'), dependency('mutter-cogl-pango-4'),
		dependency('mutter-cogl-path-4'), dependency('mutter-clutter-4')
	]
	vala_flags += ['--define', 'HAS_MUTTER330', '--define', 'HAS_MUTTER332']
	add_project_arguments(['-DCLUTTER_ENABLE_COMPOSITOR_API', '-DCLUTTER_ENABLE_EXPERIMENTAL_API',
		'-DCOGL_ENABLE_EXPERIMENTAL_API', '-DCOGL_ENABLE_EXPERIMENTAL_2_0_API'], language: 'c')
endif

mutter334_dep = dependency('libmutter-5', version: ['>= 3.34', '< 3.35.1'], required: false)
if mutter334_dep.found()
	mutter_packages_command = [
		'--pkg', 'libmutter-5',
		'--pkg', 'mutter-cogl-5',
		'--pkg', 'mutter-cogl-pango-5',
		'--pkg', 'mutter-cogl-path-5',
		'--pkg', 'mutter-clutter-5'
	]
	libmutter_dep = dependency('libmutter-5', version: '>= 3.34')
	pkgconf.set('MUTTER_PKGS', 'libmutter-5 >= 3.34')
	mutter_dep = [
		libmutter_dep,
		dependency('mutter-cogl-5'), dependency('mutter-cogl-pango-5'),
		dependency('mutter-cogl-path-5'), dependency('mutter-clutter-5')
	]
	vala_flags += ['--define', 'HAS_MUTTER330', '--define', 'HAS_MUTTER332', '--define', 'HAS_MUTTER334']
	add_project_arguments(['-DCLUTTER_ENABLE_COMPOSITOR_API', '-DCLUTTER_ENABLE_EXPERIMENTAL_API',
		'-DCOGL_ENABLE_EXPERIMENTAL_API', '-DCOGL_ENABLE_EXPERIMENTAL_2_0_API'], language: 'c')
endif

mutter336_dep = dependency('libmutter-6', version: ['>= 3.35.1', '< 3.38'], required: false)
if mutter336_dep.found()
	mutter_packages_command = [
		'--pkg', 'libmutter-6',
		'--pkg', 'mutter-cogl-6',
		'--pkg', 'mutter-cogl-pango-6',
		'--pkg', 'mutter-cogl-path-6',
		'--pkg', 'mutter-clutter-6'
	]
	libmutter_dep = dependency('libmutter-6', version: '>= 3.35.1')
	pkgconf.set('MUTTER_PKGS', 'libmutter-6 >= 3.35.1')
	mutter_dep = [
		libmutter_dep,
		dependency('mutter-cogl-6'), dependency('mutter-cogl-pango-6'),
		dependency('mutter-cogl-path-6'), dependency('mutter-clutter-6')
	]
	vala_flags += ['--define', 'HAS_MUTTER330', '--define', 'HAS_MUTTER332', '--define', 'HAS_MUTTER334', '--define', 'HAS_MUTTER336']
	add_project_arguments(['-DCLUTTER_ENABLE_COMPOSITOR_API', '-DCLUTTER_ENABLE_EXPERIMENTAL_API',
		'-DCOGL_ENABLE_EXPERIMENTAL_API', '-DCOGL_ENABLE_EXPERIMENTAL_2_0_API'], language: 'c')
endif

if mutter_dep.length() == 0
	error ('No supported mutter library found!')
endif

mutter_typelib_dir = libmutter_dep.get_pkgconfig_variable('typelibdir')

add_project_arguments(vala_flags, language: 'vala')
add_project_link_arguments(['-Wl,-rpath,@0@'.format(mutter_typelib_dir)], language: 'c')

gala_base_dep = [glib_dep, gee_dep, gtk_dep, plank_dep, bamf_dep, mutter_dep, granite_dep, gnome_desktop_dep, m_dep]

subdir('data')
subdir('lib')
subdir('src')
subdir('daemon')
subdir('plugins/maskcorners')
subdir('plugins/notify')
subdir('plugins/pip')
subdir('plugins/template')
subdir('plugins/zoom')
if get_option('documentation')
	subdir('docs')
endif
subdir('po')

meson.add_install_script('build-aux/meson/post_install.py')
