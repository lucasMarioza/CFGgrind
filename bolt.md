# BOLT
You can use CFGGrind as the profiler for [BOLT](https://github.com/facebookincubator/BOLT) optimizations, to do that you need to do the following steps

## Instalation
To install BOLT you need to follow this [section](https://github.com/facebookincubator/BOLT#installation)
You may also want to add the bin/ file to you PATH

## Compiling
BOLT requires that the program is compiled with relocs, so when compiling the program you should run 

    $ gcc

## Profiling
After having the binary, we need to profile it.

### Perf
The recommended profiler in official BOLT repo is perf, if you want to use it you can run

    $ perf record -e cycles:u -j any,u -o profile.data -- <executable>

`u` means that it will ignore kernel level calls, and `cycles` was test empirically, by BOLT developers, as the best event in this case 
  
### CFGGrind
In our case we want to use CFGGrind as profiler for BOLT, in that case we can just do as in the readme of this repo

    $ cfggrind_asmmap <executable> > cfg.map
    $ valgrind --tool=cfggrind --cfg-outfile=cfg.cfg --instrs-map=cfg.map ./<executable>

  
## Convert to BOLT Input Format
BOLT requires a specific format as input, so we need to convert it
  
### Perf
If you have used Perf in the previous step, bolt supplies a tool for this conversion, present in BOLT's bin folder
  
    $ perf2bolt -p profile.data -o profile.fdata <executable>
  
### CFGGrind
In case you have used CFGGrind, you should use cfggrind2bolt present in this repo root folder
    $ cfggrind2bolt -b <executable> cfg.cfg > profile.fdata
  
## Optimizing
After that, we are read to run bolt, so, you can just run

    $ llvm-bolt <executable> -o <executable>.bolt -report-stale -data=profile.fdata -reorder-blocks=cache+ -reorder-functions=hfsort -split-functions=2 -split-all-cold -split-eh -dyno-stats

The new executable will be named as `<executable>.bolt` where `<executable>` is the name of the original file
