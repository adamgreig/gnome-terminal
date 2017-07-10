service_conf = configuration_data()
service_conf.set('libexecdir', gt_libexecdir)

service = 'org.gnome.Terminal.service'

configure_file(
  input: service + '.in',
  output: service,
  install: true,
  install_dir: join_paths(gt_datadir, 'dbus-1', 'services'),
  configuration: service_conf
)

service = 'gnome-terminal-server.service'

configure_file(
  input: service + '.in',
  output: service,
  install: true,
  install_dir: join_paths(gt_libdir, 'systemd', 'user'),
  configuration: service_conf
)

install_data(
  'org.gnome.Terminal.gschema.xml',
  install_dir: join_paths(gt_datadir, 'glib-2.0', 'schemas')
)

gt_common_cflags = [
  '-DTERMINAL_COMPILATION'
]

# gnome-terminal-server
enum_headers = files(
  'terminal-enums.h'
)

gt_server_sources = files(
  'eggshell.c',
  'profile-editor.c',
  'server.c',
  'terminal-accels.c',
  'terminal-app.c',
  'terminal-debug.c',
  'terminal-encoding.c',
  'terminal-gdbus.c',
  'terminal-icon-button.c',
  'terminal-info-bar.c',
  'terminal-i18n.c',
  'terminal-mdi-container.c',
  'terminal-notebook.c',
  'terminal-prefs.c',
  'terminal-profiles-list.c',
  'terminal-settings-list.c',
  'terminal-screen.c',
  'terminal-screen-container.c',
  'terminal-search-popover.c',
  'terminal-tab-label.c',
  'terminal-tabs-menu.c',
  'terminal-util.c',
  'terminal-window.c'
)

enum = 'terminal-type-builtins'

terminal_type = gnome.mkenums(
  enum,
  sources: enum_headers,
  c_template: enum + '.c.template',
  h_template: enum + '.h.template'
)

service = 'org.gnome.Terminal'
namespace = 'Terminal'

terminal_gdbus = gnome.gdbus_codegen(
  'terminal-gdbus-generated',
  service + '.xml',
  interface_prefix: service,
  namespace: namespace,
  object_manager: true
)

if enable_search_provider
  gt_server_sources += files(
    'terminal-search-provider.c'
  )

  search_provider = 'terminal-search-provider-gdbus-generated'

  gt_server_sources += gnome.gdbus_codegen(
    search_provider,
    search_provider_iface,
    interface_prefix: 'org.gnome.Shell',
    namespace: namespace
  )

  install_data(
    'gnome-terminal-search-provider.ini',
    install_dir: join_paths(gt_datadir, 'gnome-shell', 'search-providers')
  )
endif

marshal = 'terminal-marshal'

gt_server_sources += gnome.genmarshal(
  marshal,
  sources: marshal + '.list',
  prefix: '_terminal_marshal',
  internal: true
)

resources_files = files(
  'preferences.ui',
  'profile-preferences.ui',
  'search-popover.ui',
  'terminal-menus.ui',
  'terminal-window.ui',
  'terminal.about',
  'terminal.common.css',
  'terminal.xml'
)

resources = 'terminal-resources'

gt_server_sources += gnome.compile_resources(
  resources,
  'terminal.gresource.xml',
  source_dir: '.',
  c_name: 'terminal',
  dependencies: resources_files,
  export: true
)

version_conf = configuration_data()
version_conf.set('TERMINAL_MAJOR_VERSION', gt_major_version)
version_conf.set('TERMINAL_MINOR_VERSION', gt_minor_version)
version_conf.set('TERMINAL_MICRO_VERSION', gt_micro_version)

version = 'terminal-version.h'

gt_server_sources += configure_file(
  input: version + '.in',
  output: version,
  configuration: version_conf
)

gt_server_cflags = gt_common_cflags + common_flags + [
  '-DTERM_LIBEXECDIR="@0@"'.format(gt_libexecdir),
  '-DTERM_LOCALEDIR="@0@"'.format(gt_localedir)
]

executable(
  gt_name + '-server',
  gt_server_sources + [terminal_gdbus, terminal_type],
  include_directories: top_inc,
  dependencies: gt_deps,
  c_args: gt_server_cflags,
  install: true,
  install_dir: gt_libexecdir
)

# Legacy terminal client
gt_sources = files(
  'terminal.c',
  'terminal-client-utils.c',
  'terminal-debug.c',
  'terminal-i18n.c',
  'terminal-options.c',
  'terminal-profiles-list.c',
  'terminal-settings-list.c'
)

gt_cflags = gt_common_cflags + common_flags + [
  '-DTERMINAL_CLIENT',
  '-DTERM_DATADIR="@0@"'.format(gt_datadir),
  '-DTERM_LOCALEDIR="@0@"'.format(gt_localedir),
  '-DTERM_PKGDATADIR="@0@"'.format(gt_pkgdatadir)
]

executable(
  gt_name,
  gt_sources + [terminal_gdbus, terminal_type],
  include_directories: top_inc,
  dependencies: gt_deps,
  c_args: gt_cflags,
  install: true,
  install_dir: gt_bindir
)

# Checks
test_name = 'terminal-regex'

exe = executable(
  test_name,
  test_name + '.c',
  include_directories: top_inc,
  c_args: [
    '-DTERMINAL_REGEX_MAIN'
  ],
  dependencies: gt_deps
)

test(test_name, exe)

# Terminal client
if enable_gterminal
  gterminal_sources = files(
    'gterminal.vala',
    'client.vapi',
    'config.vapi',
    'profiles.vapi',
    'terminal-client-utils.c',
    'terminal-debug.c',
    'terminal-profiles-list.c',
    'terminal-settings-list.c'
  )

  gterminal_cflags = gt_common_cflags + [
    '-DTERMINAL_CLIENT',
    '-DLOCALEDIR="@0@"'.format(gt_localedir),
    '-DGETTEXT_PACKAGE="@0@"'.format(gt_name)
  ]

  # See bug #710862 about -Wsuggest-attribute=format
  test_cflags = [
    '-Wno-cast-qual',
    '-Wno-format-nonliteral',
    '-Wno-suggest-attribute=format',
    '-Wno-unused-but-set-variable',
    '-Wno-unused-function',
    '-Wno-unused-variable',
    '-Wno-write-strings'
  ]

  foreach cflag: test_cflags
    if cc.has_argument(cflag)
      gterminal_cflags += [cflag]
    endif
  endforeach

  executable(
    'gterminal',
    gterminal_sources + [terminal_type],
    include_directories: top_inc,
    dependencies: gterminal_deps,
    # FIXME: uuid include path should be removed
    c_args: gterminal_cflags + [
      '-I/usr/include/uuid'
    ],
    vala_args: [
      '--pkg=posix'
    ],
    # FIXME: uuid lib should be removed
    link_args: '-luuid'
  )
endif

# Pref migrator
if enable_migration
  gt_migration_sources = files(
    'migration.c',
    'terminal-debug.c',
    'terminal-profiles-list.c',
    'terminal-settings-list.c'
  )

  executable(
    gt_name + '-migration',
    gt_migration_sources + [terminal_type],
    include_directories: top_inc,
    dependencies: gt_migration_deps,
    c_args: common_flags,
    install: true,
    install_dir: gt_libexecdir
  )
endif

# Nautilus extension
if enable_nautilus_extension
  nautilus_sources = files(
    'terminal-client-utils.c',
    'terminal-i18n.c',
    'terminal-nautilus.c'
  )

  symbol_map = 'nautilus.map'

  ldflag = '-Wl,--version-script,@0@/@1@'.format(meson.current_source_dir(), symbol_map)

  nautilus_ldflags = []
  if host_machine.system().contains('linux')
    if cc.has_argument(ldflag)
      nautilus_ldflags += ldflag
    endif
  endif

  shared_module(
    'terminal-nautilus',
    sources: nautilus_sources + [terminal_gdbus, terminal_type],
    include_directories: top_inc,
    dependencies: nautilus_deps,
    c_args: [
      '-DTERM_LOCALEDIR="@0@"'.format(gt_localedir)
    ],
    link_args: nautilus_ldflags,
    link_depends: symbol_map,
    install: true,
    install_dir: nautilus_dir
  )
endif
