#!python
import os, subprocess

# Reads variables from an optional file.
customs = ['../custom.py']
opts = Variables(customs, ARGUMENTS)

# Gets the standard flags CC, CCX, etc.
env = DefaultEnvironment()

# Define our parameters
opts.Add(EnumVariable('platform', "Platform", 'windows', ['linux', 'osx', 'windows']))
opts.Add(EnumVariable('target', "Compilation target", 'release', ['d', 'debug', 'r', 'release']))
opts.AddVariables(
    PathVariable('target_path', 'The path where the lib is installed.', 'demo/addons/godot-openxr/bin/'),
    PathVariable('target_name', 'The library name.', 'godot_openxr', PathVariable.PathAccept),
)
opts.Add(BoolVariable('use_llvm', "Use the LLVM / Clang compiler", 'no'))

# Updates the environment with the option variables.
opts.Update(env)

# Paths
godot_glad_path = "glad/"
godot_headers_path = "godot_headers/"
target_path = env['target_path']

openxr_include_path = ""
openxr_library_path = ""

# Source files to include
sources = []

# Platform dependent settings
if env['platform'] == "windows":
    target_path += "win64/"
    openxr_include_path += "openxr_loader_windows/1.0.9/include/"
    openxr_library_path += "openxr_loader_windows/1.0.9/x64/lib"

    # Check some environment settings
    if env['use_llvm']:
        env['CXX'] = 'clang++'
        env['CC'] = 'clang'

        if env['target'] in ('debug', 'd'):
            env.Append(CCFLAGS = ['-fPIC', '-g3','-Og', '-std=c++17'])
        else:
            env.Append(CCFLAGS = ['-fPIC', '-g','-O3', '-std=c++17'])
    else:
        # This makes sure to keep the session environment variables on windows,
        # that way you can run scons in a vs 2017 prompt and it will find all the required tools
        env.Append(ENV = os.environ)

        env.Append(CCFLAGS = ['-DWIN32', '-D_WIN32', '-D_WINDOWS', '-W3', '-GR', '-D_CRT_SECURE_NO_WARNINGS','-std:c++latest'])
        if env['target'] in ('debug', 'd'):
            env.Append(CCFLAGS = ['-EHsc', '-D_DEBUG', '/MTd'])
        else:
            env.Append(CCFLAGS = ['-O2', '-EHsc', '-DNDEBUG', '/MT'])

    # Do we need these?
    env.Append(LIBS = ["opengl32", "setupapi", "advapi32.lib"])

    # For now just include Glad on Windows, but maybe also use with Linux?
    env.Append(CPPPATH = [godot_glad_path])
    sources += Glob(godot_glad_path + '*.c')

elif env['platform'] == "linux":
    target_path += "linux/"

    # note, on linux the OpenXR SDK is installed in /usr and should be accessible
    if env['use_llvm']:
        env['CXX'] = 'clang++'
        env['CC'] = 'clang'

    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS = ['-fPIC', '-ggdb','-O0'])
    else:
        env.Append(CCFLAGS = ['-fPIC', '-g','-O3'])
    env.Append(CXXFLAGS = [ '-std=c++0x' ])
    env.Append(LINKFLAGS = [ '-Wl,-R,\'$$ORIGIN\'' ])

#elif env['platform'] == "osx":
#    # not tested
#
#    target_path += "win64/"
#    openxr_include_path += "??"
#    openxr_library_path += "??"
#
#    env.Append(CCFLAGS = ['-g','-O3', '-arch', 'x86_64'])
#    env.Append(LINKFLAGS = ['-arch', 'x86_64'])
#    env.Append(LINKFLAGS=['-framework', 'Cocoa', '-framework', 'OpenGL', '-framework', 'IOKit'])
#    env.Append(LIBS=['pthread'])


####################################################################################################################################
# and add our main project

env.Append(CPPPATH=['.', 'src/', godot_headers_path])

if openxr_include_path != "":
    env.Append(CPPPATH = [ openxr_include_path ])

if openxr_library_path != "":
    env.Append(LIBPATH = [ openxr_library_path ])

env.Append(LIBS=['openxr_loader'])

sources += Glob('src/*.c')
sources += Glob('src/*.cpp')
sources += Glob('src/*/*.c')
sources += Glob('src/*/*.cpp')

library = env.SharedLibrary(target=target_path + env['target_name'], source=sources)
Default(library)

# Generates help for the -h scons option.
Help(opts.GenerateHelpText(env))
