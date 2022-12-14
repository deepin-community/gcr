version_xml = configure_file(
  input: 'version.xml.in',
  output: '@BASENAME@',
  configuration: { 'VERSION': gcr_version },
)

gcr_docs_images = [
  'images/certificate-widget.png',
  'images/combo-selector.png',
  'images/import-button.png',
  'images/list-selector.png',
  'images/key-widget.png',
  'images/tree-selector.png',
  'images/viewer-widget.png',
]

gcr_docs_ignore_headers = [
  'console-interaction.h',
  'eggimagemenuitem.h',
  'gcr-callback-output-stream.h',
  'gcr-certificate-exporter.h',
  'gcr-certificate-basics-widget.h',
  'gcr-certificate-details-widget.h',
  'gcr-certificate-request-renderer.h',
  'gcr-dbus-constants.h',
  'gcr-dbus-generated.h',
  'gcr-deprecated.h',
  'gcr-deprecated-base.h',
  'gcr-display-scrolled.h',
  'gcr-display-view.h',
  'gcr-failure-renderer.h',
  'gcr-gnupg-importer.h',
  'gcr-gnupg-renderer.h',
  'gcr-icons.h',
  'gcr-import-dialog.h',
  'gcr-internal.h',
  'gcr-live-search.h',
  'gcr-marshal.h',
  'gcr-menu-button.h',
  'gcr-oids.h',
  'gcr-openpgp.h',
  'gcr-pkcs11-importer.h',
  'gcr-pkcs11-import-interaction.h',
  'gcr-pkcs11-import-dialog.h',
  'gcr-pkcs11-renderer.h',
  'gcr-record.h',
  'gcr-single-collection.h',
  'gcr-unlock-renderer.h',
  'gcr-viewer-window.h',
  'gcr-xxx.h',
  'gcr-zzz.h',
]

gnome.gtkdoc('gcr',
  main_sgml: 'gcr-docs.sgml',
  content_files: files(
    'gcr-visual-index.xml',
  ),
  src_dir: [
    include_directories('../../../gcr'),
    include_directories('../../../ui'),
  ],
  ignore_headers: gcr_docs_ignore_headers,
  dependencies: [ gcr_base_dep, gcr_ui_dep ],
  gobject_typesfile: 'gcr.types',
  html_assets: gcr_docs_images,
  scan_args: [
    '--rebuild-types',
    '--deprecated-guards=GCR_DISABLE_DEPRECATED',
  ],
  install: true,
)

install_data(gcr_docs_images,
  install_dir: gnome.gtkdoc_html_dir('gcr'),
)
