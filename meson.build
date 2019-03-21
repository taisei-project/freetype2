project('freetype2', 'c',
    license : ['FTL', 'GPL2'],
    meson_version: '>= 0.45.0',
    version: '2.10.1',
    default_options: ['buildtype=debugoptimized']
)

# Based on https://github.com/mesonbuild/freetype2/blob/2.9.1/meson.build

# NOTE about Freetype versions
# There are 3 versions numbers associated with each releases:
# - official release number (eg. 2.9.1) - accessible via FREETYPE_{MAJOR,MINOR,PATCH} macros from FT_FREETYPE_H
# - libtool-specific version number, this is hat is returned by freetype-config --version / pkg-config --modversion (eg. 22.1.16)
# - the platform-specific shared object version number (eg. 6.16.1)
# See https://git.savannah.gnu.org/cgit/freetype/freetype2.git/tree/docs/VERSIONS.TXT
# for more information
release_version = meson.project_version()
libtool_version = '22.1.16'
so_version = '6.16.1'
so_soversion = '6'

cc = meson.get_compiler('c')

base_sources = [
    'src/autofit/autofit.c',
    'src/base/ftbase.c',
    'src/base/ftbbox.c',
    'src/base/ftbdf.c',
    'src/base/ftbitmap.c',
    'src/base/ftcid.c',
    'src/base/ftfstype.c',
    'src/base/ftgasp.c',
    'src/base/ftglyph.c',
    'src/base/ftgxval.c',
    'src/base/ftinit.c',
    'src/base/ftmm.c',
    'src/base/ftotval.c',
    'src/base/ftpatent.c',
    'src/base/ftpfr.c',
    'src/base/ftstroke.c',
    'src/base/ftsynth.c',
    'src/base/ftsystem.c',
    'src/base/fttype1.c',
    'src/base/ftwinfnt.c',
    'src/bdf/bdf.c',
    'src/bzip2/ftbzip2.c',
    'src/cache/ftcache.c',
    'src/cff/cff.c',
    'src/cid/type1cid.c',
    'src/gzip/ftgzip.c',
    'src/lzw/ftlzw.c',
    'src/pcf/pcf.c',
    'src/pfr/pfr.c',
    'src/psaux/psaux.c',
    'src/pshinter/pshinter.c',
    'src/psnames/psnames.c',
    'src/raster/raster.c',
    'src/sfnt/sfnt.c',
    'src/smooth/smooth.c',
    'src/truetype/truetype.c',
    'src/type1/type1.c',
    'src/type42/type42.c',
    'src/winfonts/winfnt.c',
]

if host_machine.system() == 'windows'
    winmod = import('windows')
    base_sources += [
        'builds/windows/ftdebug.c',
        winmod.compile_resources('src/base/ftver.rc'),
    ]
else
    base_sources += [
        'src/base/ftdebug.c',
    ]
endif

c_args = [
    '-DFT2_BUILD_LIBRARY',
    '-DFT_CONFIG_CONFIG_H=<ftconfig.h>',
    '-DFT_CONFIG_OPTIONS_H=<ftoption.h>'
]

c_args += cc.get_supported_arguments('-Wcast-function-type')

if cc.get_id() == 'msvc'
    c_args += [
        '-D_CRT_SECURE_NO_WARNINGS',
        '-D_CRT_NONSTDC_NO_WARNINGS',
    ]
endif

check_headers = []
have_posix = cc.has_header_symbol('unistd.h', '_POSIX_VERSION')

check_headers += [
    'unistd.h',
    'fcntl.h',
    'stdint.h',
]

if have_posix
    ftconfig_h_in = files('builds/unix/ftconfig.h.in')
else
    ftconfig_h_in = files('include/freetype/config/ftconfig.h')
endif

conf = configuration_data()
deps = []
incbase = include_directories(['include'])

foreach name : check_headers
    if cc.has_header(name)
        conf.set('HAVE_@0@'.format(name.to_upper().underscorify()), 1)
    endif
endforeach

zlib_dep = dependency('zlib', fallback : ['zlib', 'zlib_dep'])
conf.set('FT_CONFIG_OPTION_SYSTEM_ZLIB', 1)
deps += [zlib_dep]

if get_option('use_asm')
    conf.set('FT_CONFIG_OPTION_INLINE_MULFIX', false)
else
    conf.set('FT_CONFIG_OPTION_NO_ASSEMBLER', true)
endif

configure_file(
    input : ftconfig_h_in,
    output : 'ftconfig.h',
    configuration : conf,
)

configure_file(
    input : 'include/freetype/config/ftoption.h.in',
    output : 'ftoption.h',
    configuration : conf,
)

libfreetype = library('freetype', base_sources,
    include_directories : incbase,
    dependencies : deps,
    c_args : c_args,
    install : false,
    build_by_default : false,
    version : so_version,
    soversion : so_soversion
)

freetype_dep = declare_dependency(
    link_with : libfreetype,
    include_directories : incbase,
    dependencies : deps
)
