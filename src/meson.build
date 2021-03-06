src_inc = include_directories('.')

cflags = [
  '-DEOG_DATA_DIR="@0@"'.format(eog_datadir),
  '-DEOG_LOCALE_DIR="@0@"'.format(eog_localedir)
]

enum_headers = files(
  'eog-scroll-view.h',
  'eog-window.h'
)

gnome.mkenums(
  'org.gnome.eog.enums.xml',
  sources: enum_headers,
  comments: '<!-- @comment@ -->',
  fhead: '<schemalist>',
  vhead: '  <@type@ id="org.gnome.eog.@EnumName@">',
  vprod: '    <value nick="@valuenick@" value="@valuenum@"/>',
  vtail: '  </@type@>',
  ftail: '</schemalist>',
  install_header: true,
  install_dir: eog_schemadir
)

headers = enum_headers + files(
  'eog-application.h',
  'eog-application-activatable.h',
  'eog-clipboard-handler.h',
  'eog-debug.h',
  'eog-enums.h',
  'eog-file-chooser.h',
  'eog-image.h',
  'eog-image-save-info.h',
  'eog-job-scheduler.h',
  'eog-jobs.h',
  'eog-list-store.h',
  'eog-properties-dialog.h',
  'eog-sidebar.h',
  'eog-statusbar.h',
  'eog-thumbnail.h',
  'eog-thumb-nav.h',
  'eog-thumb-view.h',
  'eog-transform.h',
  'eog-uri-converter.h',
  'eog-window-activatable.h'
)

sources = files(
  'eog-application.c',
  'eog-application-activatable.c',
  'eog-clipboard-handler.c',
  'eog-close-confirmation-dialog.c',
  'eog-debug.c',
  'eog-error-message-area.c',
  'eog-file-chooser.c',
  'eog-image.c',
  'eog-image-jpeg.c',
  'eog-image-save-info.c',
  'eog-job-scheduler.c',
  'eog-jobs.c',
  'eog-list-store.c',
  'eog-metadata-sidebar.c',
  'eog-metadata-reader.c',
  'eog-metadata-reader-jpg.c',
  'eog-metadata-reader-png.c',
  'eog-pixbuf-util.c',
  'eog-plugin-engine.c',
  'eog-preferences-dialog.c',
  'eog-print.c',
  'eog-print-image-setup.c',
  'eog-print-preview.c',
  'eog-properties-dialog.c',
  'eog-save-as-dialog-helper.c',
  'eog-scroll-view.c',
  'eog-session.c',
  'eog-sidebar.c',
  'eog-statusbar.c',
  'eog-thumbnail.c',
  'eog-thumb-nav.c',
  'eog-thumb-view.c',
  'eog-transform.c',
  'eog-uri-converter.c',
  'eog-util.c',
  'eog-window-activatable.c',
  'eog-window.c',
  'eog-zoom-entry.c',
  'zoom.c'
)

if enable_libexif
  headers += files('eog-exif-util.h')
  sources += files('eog-exif-util.c')
endif

if enable_libexif or enable_exempi
  sources += files('eog-metadata-details.c')
endif

install_headers(
  headers,
  subdir: join_paths(eog_api_name, meson.project_name())
)

sources += gnome.compile_resources(
  'eog-resources',
  'eog.gresource.xml',
  source_dir: data_dir,
  c_name: '_eog',
  dependencies: resource_data,
  export: true
)

enum = 'eog-enum-types'

sources += gnome.mkenums(
  enum,
  sources: headers,
  c_template: enum + '.c.template',
  h_template: enum + '.h.template'
)

marshal = 'eog-marshal'

sources += gnome.genmarshal(
  marshal,
  sources: marshal + '.list',
  prefix: 'eog_marshal',
  internal: true
)

deps = eog_deps

libeog_cflags = cflags + [
  '-DEOG_PLUGIN_DIR="@0@"'.format(eog_pluginsdir),
  '-DEOG_PREFIX="@0@"'.format(eog_prefix),
  '-DG_LOG_DOMAIN="@0@"'.format(meson.project_name().to_upper()),
  '-DLIBDIR="@0@"'.format(eog_libdir)
]

symbol_map = join_paths(meson.current_source_dir(), 'symbol.map')

ldflags = []
if have_version_script
  ldflags += '-Wl,--version-script,@0@'.format(symbol_map)
endif

if enable_libjpeg
  deps += libjpegutils_dep
endif

libeog = shared_library(
  meson.project_name(),
  sources: sources,
  include_directories: top_inc,
  dependencies: deps,
  c_args: libeog_cflags,
  link_args: ldflags,
  link_depends: symbol_map,
  install: true,
  install_dir: eog_pkglibdir
)

libeog_dep = declare_dependency(
  link_with: libeog,
  include_directories: src_inc,
  dependencies: eog_deps
)

pkg.generate(
  libraries: libeog,
  version: eog_version,
  name: 'Eye of GNOME',
  description: 'The GNOME Image Viewer',
  filebase: meson.project_name(),
  subdirs: eog_api_name,
  requires: 'gtk+-3.0',
  variables: [
    'exec_prefix=' + eog_prefix,
    'pluginsdir=' + eog_pluginsdir
  ]
)

eog = executable(
  meson.project_name(),
  'main.c',
  include_directories: top_inc,
  dependencies: libeog_dep,
  c_args: cflags,
  install: true,
  install_rpath: eog_pkglibdir
)

if enable_introspection
  gir_sources = sources + headers

  gir_dir = join_paths(eog_pkgdatadir, 'gir-' + eog_gir_version)
  typelib_dir = join_paths(eog_pkglibdir, 'girepository-' + eog_gir_version)

  gnome.generate_gir(
    eog,
    sources: gir_sources,
    nsversion: eog_api_version,
    namespace: eog_gir_ns,
    includes: 'Gtk-3.0',
    install: true,
    install_dir_gir: gir_dir,
    install_dir_typelib: typelib_dir
  )
endif
