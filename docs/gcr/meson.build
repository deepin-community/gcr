gcr_base_content_files = [
  'non-pageable-memory.md',
  'pkcs11-configuration.md',
  'trust-store.md',
]

gcr_base_toml = configure_file(
  input: 'gcr-base.toml.in',
  output: '@BASENAME@',
  configuration: {
    'GCR_VERSION': gcr_major_version,
    'GCK_VERSION': gck_major_version,
  },
)

gcr_base_docs = custom_target('gcr-docs',
  input: gcr_gir[0],
  output: 'gcr-@0@'.format(gcr_major_version),
  command: [
    gi_docgen,
    'generate',
    '--quiet',
    '--fatal-warnings',
    '--add-include-path=@0@'.format(build_root / 'gck'),
    '--config', gcr_base_toml,
    '--output-dir=@OUTPUT@',
    '--no-namespace-dir',
    '--content-dir=@0@'.format(meson.current_source_dir()),
    '@INPUT@',
  ],
  depend_files: [ gcr_base_toml, gcr_base_content_files ],
  depends: [ gck_gir[0] ],
  build_by_default: true,
  install: true,
  install_dir: get_option('datadir') / 'doc',
)
