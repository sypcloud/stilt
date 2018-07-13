---
layout: docs
title: STILT Docs
---

# STILT Docs

This STILT documentation is intended for the modified Stochastic Time-Inverted Lagrangian Transport (STILT) model that includes an improved interface wrapper and enhancements improving the model's applicability to fine-scale measurement and flux interpretation.  

[Model source code](https://github.com/uataq/stilt), [tutorials](https://github.com/uataq/stilt-tutorials), information [about STILT]({{"/"|relative_url}}), and [documentation for model configuration]({{"/docs"|relative_url}}) are all freely available.

Confused? Take a step back and check out the [About]({{"/about"|relative_url}}) page.

### Relevant manuscript

Fasoli, B., Lin, J. C., Bowling, D. R., Mitchell, L., and Mendoza, D.: Simulating atmospheric tracer concentrations for spatially distributed receptors: updates to the Stochastic Time-Inverted Lagrangian Transport model's R interface (STILT-R version 2), Geosci. Model Dev., 11, 2813-2824, [10.5194/gmd-11-2813-2018](https://doi.org/10.5194/gmd-11-2813-2018), 2018.


# Quick Start

Familiar with the workflow and ready to start a new STILT project?

1. Initialize project with `Rscript -e "uataq::stilt_init('myproject')"`
    <div class="terminal">
      <div class="terminal-osx-button"></div>
      <div class="terminal-osx-button"></div>
      <div class="terminal-osx-button"></div>
      <div class="terminal-command">
        Rscript -e "uataq::stilt_init('myproject')"
      </div>
      <div class="terminal-return">
        Cloning into 'stilt'...<br>
        remote: Counting objects: 1712, done.<br>
        remote: Compressing objects: 100% (275/275), done.<br>
        remote: Total 1712 (delta 71), reused 267 (delta 39), pack-reused 1384<br>
        Receiving objects: 100% (1712/1712), 32.85 MiB | 8.57 MiB/s, done.<br>
        Resolving deltas: 100% (652/652), done.<br>
        Checking connectivity... done.<br>
        gfortran  -fpic -g -O2 -fstack-protector --param=ssp-buffer-size=4  -c  permute.f90 -o permute.o<br>
        gfortran -shared -L/usr/lib/R/lib -Wl,-Bsymbolic-functions -Wl,-z,relro -o permute.so permute.o -L/usr/lib/R/lib -lR<br>
        gfortran -c -O2 -fconvert=big-endian -frecord-marker=4 -fbounds-check -ffree-form -I. funits.f<br>
        ...    
      </div>
    </div>

2. Edit settings in `r/run_stilt.r`
    <div class="terminal">
      <div class="terminal-osx-button"></div>
      <div class="terminal-osx-button"></div>
      <div class="terminal-osx-button"></div>
      <div class="terminal-command">
        cd myproject
      </div>
      <div class="terminal-command">
        ls
      </div>
      <div class="terminal-return">
        exe  fortran  out  r  README.md  setup  stilt
      </div>
      <div class="terminal-command">
        vim r/run_stilt.r
      </div>
      <div class="terminal-return">
        ...
      </div>
    </div>

3. Run model with `Rscript r/run_stilt.r`
    <div class="terminal">
      <div class="terminal-osx-button"></div>
      <div class="terminal-osx-button"></div>
      <div class="terminal-osx-button"></div>
      <div class="terminal-command">
        Rscript r/run_stilt.r
      </div>
      <div class="terminal-return">
        Initializing STILT<br>
        Number of receptors: 1<br>
        Number of parallel threads: 1<br>
        Estimated footprint grid RAM allocation: 314 MB<br>
        Parallelization disabled. Executing simulations sequentially...<br>
        <br>
        Running simulation ID:   2015061822_-111.980323_40.782561_5
      </div>
    </div>

To get started, see [Installation]({{"/docs/install.html"|relative_url}}).
