gck_headers_install_dir = 'gck-@0@'.format(gck_major_version) / 'gck'

gck_sources_public = files(
  'gck-attributes.c',
  'gck-dump.c',
  'gck-enumerator.c',
  'gck-misc.c',
  'gck-module.c',
  'gck-modules.c',
  'gck-object.c',
  'gck-object-cache.c',
  'gck-password.c',
  'gck-session.c',
  'gck-slot.c',
  'gck-uri.c',
)

gck_sources_private = files(
  'gck-call.c',
  'gck-interaction.c',
)

gck_headers = files(
  'gck.h',
)

# Generated sources
gck_version_conf = {
  'GCR_MAJOR': gck_major_version,
  'GCR_MINOR': gck_minor_version,
  'GCR_MICRO': gck_micro_version,
}
gck_version_h = configure_file(input: 'gck-version.h.in',
  output: 'gck-version.h',
  configuration: gck_version_conf,
)

gck_enums_gen = gnome.mkenums_simple('gck-enum-types',
  sources: gck_headers,
  install_header: true,
  install_dir: get_option('includedir') / gck_headers_install_dir,
)

gck_marshal_gen = gnome.genmarshal('gck-marshal',
  sources: 'gck-marshal.list',
  prefix: '_gck_marshal',
)

gck_sources = [
  gck_sources_public,
  gck_sources_private,
  gck_enums_gen,
  gck_marshal_gen,
]

gck_exported_headers = [
  gck_headers,
  gck_version_h,
  'gck-deprecated.h',
  'pkcs11.h',
  'pkcs11n.h',
  'pkcs11x.h',
]

install_headers(gck_exported_headers,
  subdir: gck_headers_install_dir,
)

gck_deps = [
  glib_deps,
  p11kit_dep,
  libegg_dep,
]

gck_cflags = [
  '-DG_LOG_DOMAIN="@0@"'.format('Gck'),
  '-DGCK_API_SUBJECT_TO_CHANGE',
  '-DP11_KIT_API_SUBJECT_TO_CHANGE',
  '-DPKCS11_REGISTRY_DIR="@0@"'.format(get_option('prefix') / get_option('libdir') / 'pkcs11'),
  '-D_XOPEN_SOURCE',   # Needed for strptime()
  '-D_DEFAULT_SOURCE', # Needed for timegm
]

gck_symbolmap = meson.current_source_dir() / 'libgck.map'
gck_linkflags = cc.get_supported_link_arguments(
  '-Wl,--version-script,@0@'.format(gck_symbolmap),
)

gck_lib = shared_library('gck-@0@'.format(gck_major_version),
  gck_sources,
  dependencies: gck_deps,
  c_args: gck_cflags,
  link_args: gck_linkflags,
  link_depends: gck_symbolmap,
  include_directories: config_h_dir,
  version: gck_soversion,
  install: true,
)

gck_dep = declare_dependency(
  link_with: gck_lib,
  include_directories: include_directories('..'),
  sources: gck_enums_gen[1], # Make sure gck-enum-types.h can be included
)

# We create an extra version of GCK which exposes internal functions
# (for testing purposes)
gck_testable_lib = library('gck-testable',
  gck_sources,
  dependencies: gck_deps,
  c_args: gck_cflags,
  include_directories: config_h_dir,
)

gck_testable_dep = declare_dependency(
  link_with: gck_testable_lib,
  include_directories: include_directories('..'),
  sources: gck_enums_gen[1], # Make sure gck-enum-types.h can be included
)

# GObject introspection (GIR) file
gck_gir_sources = [
  gck_sources_public,
  gck_headers,
  gck_enums_gen[1],
  gck_version_h,
]

if get_option('introspection')
  gck_gir = gnome.generate_gir(gck_lib,
    sources: gck_gir_sources,
    namespace: 'Gck',
    nsversion: '@0@'.format(gck_major_version),
    export_packages: 'gck-@0@'.format(gck_major_version),
    includes: [ 'GObject-2.0', 'Gio-2.0' ],
    header: 'gck/gck.h',
    extra_args: [
      '-DGCK_COMPILATION',
      '-DGCK_API_SUBJECT_TO_CHANGE',
    ],
    install: true,
  )

  gck_vapi = gnome.generate_vapi('gck-@0@'.format(gck_major_version),
    sources: gck_gir[0],
    metadata_dirs: meson.current_source_dir(),
    packages: [ 'glib-2.0', 'gio-2.0' ],
    install: true,
  )

  install_data('pkcs11.vapi',
    install_dir: get_option('datadir') / 'vala' / 'vapi',
  )
endif

# pkg-config file
pkgconfig.generate(gck_lib,
  description: 'GObject bindings for PKCS#11',
  requires: [ glib_dep, gio_dep, gobject_dep, p11kit_dep ],
  subdirs: 'gck-@0@'.format(gck_major_version),
)

# tests
gck_test_lib = library('gck-test',
  sources: [ 'gck-mock.c', 'gck-test.c' ],
  dependencies: [ gck_deps, gck_testable_dep ],
  c_args: gck_cflags,
  include_directories: config_h_dir,
)

gck_mock_test_lib = shared_library('mock-test-module',
  sources: 'mock-test-module.c',
  link_with: gck_test_lib,
  dependencies: [ gck_deps, gck_dep ],
  c_args: gck_cflags,
  include_directories: config_h_dir,
)

gck_test_names = [
  'attributes',
  'module',
  'slot',
  'session',
  'object',
  'crypto',
  'uri',
  'enumerator',
  'modules',
]

gck_test_cflags = [
  '-D_GCK_TEST_MODULE_PATH="@0@"'.format(gck_mock_test_lib.full_path()),
]

foreach _test : gck_test_names
  _test_name = 'test-gck-'+_test

  gck_test_bin = executable(_test_name,
    '@0@.c'.format(_test_name),
    link_with: [ gck_test_lib, egg_test_lib ],
    dependencies: [ gck_deps, gck_testable_dep ],
    c_args: gck_cflags + gck_test_cflags,
    include_directories: config_h_dir,
  )

  test(_test, gck_test_bin,
    suite: 'gck',
    depends: gck_mock_test_lib,
  )
endforeach
