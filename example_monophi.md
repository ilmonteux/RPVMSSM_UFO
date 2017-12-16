# mono&phi; example

Generate events in Madgraph as done for 1712.04939, 1707.05783:
```
# first define process
import model RPVMSSM_UFO
define p = u d s c b u~ d~ s~ c~ b~ g
generate p p >  ur, ur > u n1
add process p p > ur~, ur~ > u~ n1
output monophi_example
```

```
# now run madevent
launch
# do not turn on pythia/delphes...
done
# import param_card at benchmark point (1250,900), set number of events per run
param_card.monophi.dat
set nevents 10000
done
```
Note that even though the RPV couplings in param_card.monophi.dat are 0.1 by default, the production cross section (times branching ratio) is a free parameter in the likelihood evaluation (through the cross section multiplier &mu;). For example, at (1250,900) the best-fit cross section multiplier is <hat>&mu;</hat>=2.87, which (with 100% BR of &phi;&rarr;u&psi;) corresponds to &lambda;<sub>112</sub>=0.17.

For scans over parameter space, one will want to modify the masses at each step.
A small subtlety is that the UFO file has hardcoded the SUSY couplings for the squark-quark-neutralino (dependent on g<sub>1,2</sub>), but in our approach that coupling could be larger. Furthermore, we are only interested in the cross section times branching ratio into this signal. As explained in the [README](./README.md), cross sections reported by Madgraph could be wrong if the total width stored in the card is not the same as the width that it calculates for this decay chain. 
A simple workaround is to set the total width of the squark to the partial width to quark-neutralino, which in turn can be computed by Madgraph by temporarily setting the RPV coupling to zero (or, one can also compute the partial width analytically and use `set width 2000002 <analytical_width>`).
```
launch
# do not turn on pythia/delphes...
done
set nevents 10000
param_card.monophi.dat
# new masses
set mass 2000002 1500.0
set mass 1000022 1200.0
set width 1000022 0.0
# set RPV couplings to zero, compute width
set RLDD1x1x2  0.
set RLDD1x2x1  0.
compute_widths 2000002 --body_decay=2.1
#reset RPV couplings to 0.1
set RLDD1x1x2  0.1
set RLDD1x2x1  -0.1
done
```
Another workaround would be to only generate the resonant production `p p >  ur`, `p p >  ur~` and let Pythia8 decay the squark via flat matrix elements. In this case one only needs to include `pythia.readstring("2000002:oneChannel = on 1.0 102  2 1000022")` and just change the masses, as Pythia will overwrite the squark decays. The resulting events do not have spin correlations, but it is a small difference.
