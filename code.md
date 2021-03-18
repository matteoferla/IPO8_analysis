## Code

The base scripts (`pyrosetta_helper` below) 
can be found in [matteoferla/pyrosetta_scripts GitHub repo](https://github.com/matteoferla/pyrosetta_scripts).

```python
import pyrosetta
from init_boilerplate import make_option_string
pyrosetta.distributed.maybe_init(extra_options=make_option_string(no_optH=False,
                                                ex1=None,
                                                ex2=None,
                                                mute='all',
                                                ignore_unrecognized_res=True,
                                                load_PDB_components=False,
                                                ignore_waters=False)
                               )
```
Parameterise cofactor.
```python
from rdkit_to_params import Params
smiles = 'c1nc2c(n1[C@H]3[C@@H]([C@@H]([C@H](O3)CO[P@@](=O)([O-])O[P@](=O)([O-])OP(=O)([O-])[O-])O)O)[nH]c(nc2=O)N'
params = Params.from_smiles_w_pdbfile(pdb_file='IPO8.pdb',
                                        smiles=smiles,
                                        name='GTP',
                                        proximityBonding=True)
params.dump('GTP.params')
```

Import pose
```python
from pyrosetta_help.common_ops import pose_from_file
pose = pose_from_file('IPO8-RAN.r.pdb', ['GTP.params'])
```

### Relax
First restrain to prevent it from blowing up (Swissmodel...)

```python
scorefxn_cart = pyrosetta.create_score_function('ref2015_cart')
# Keep CA fixed by 5. 2.±0.5
all_sele = pyrosetta.rosetta.core.select.residue_selector.TrueResidueSelector()
constraint = pyrosetta.rosetta.protocols.constraint_generator.AtomPairConstraintGenerator()
constraint.set_residue_selector(all_sele)
constraint.set_ca_only(True)
constraint.set_use_harmonic_function(True)
constraint.set_max_distance(2.0)
constraint.set_sd(0.5)
add_csts = pyrosetta.rosetta.protocols.constraint_generator.AddConstraints()
add_csts.add_generator(constraint)
add_csts.apply(pose)
stm = pyrosetta.rosetta.core.scoring.ScoreTypeManager()
scorefxn_cart.set_weight(stm.score_type_from_name("atom_pair_constraint"), 10)
# only first chain ('A')
chain_sele = pyrosetta.rosetta.core.select.residue_selector.ChainSelector(1)
chain_vector = chain_sele.apply(pose)
movemap = pyrosetta.MoveMap()
movemap.set_bb(allow_bb=chain_vector)
movemap.set_chi(allow_chi=chain_vector)
# thunderbirds are go
relax = pyrosetta.rosetta.protocols.relax.FastRelax(scorefxn_cart, 5)
relax.set_movemap(movemap)
relax.minimize_bond_angles(True)
relax.minimize_bond_lengths(True)
relax.cartesian(True)
relax.apply(pose)

pose.dump_scored_pdb('IPO8-RAN.cart.pdb', scorefxn_cart)
```
then regualar FastRelax.
```python
scorefxn = pyrosetta.create_score_function('ref2015')
relax = pyrosetta.rosetta.protocols.relax.FastRelax(scorefxn, 15)
relax.apply(pose)
    
pose.dump_scored_pdb('IPO8-RAN.r.pdb', scorefxn)
```

### Interface score
```python
pyrosetta.rosetta.core.pose.remove_nonprotein_residues(pose)
wild_inter = {'complex_energy': ia.get_complex_energy(),
                'separated_interface_energy': ia.get_separated_interface_energy(),
                'complexed_sasa': ia.get_complexed_sasa(),
                'crossterm_interface_energy': ia.get_crossterm_interface_energy(),
                'interface_dG': ia.get_interface_dG(),
                'interface_delta_sasa': ia.get_interface_delta_sasa()}
```
`wild_inter` is:

    {'complex_energy': -3722.0121005113037,
     'separated_interface_energy': -118.99688467287115,
     'complexed_sasa': 50825.90028005577,
     'crossterm_interface_energy': -118.99688467245424,
     'interface_dG': -118.99688467287115,
     'interface_delta_sasa': 4888.5267940089725}
     
### Exon deletion

Rosetta remodel in Pyrosetta was used to model the exon deletion (447-476).
Unlike dihedral FastRelax (with jump True), the termini are fixed, so regardless of the change the un-remodelled parts will
stay put. Consequently, 441-446 were remodelled (same AA), 447-476

```python
from pyrosetta_help.blueprint_maker import Blueprinter

bluprint_filename = 'del447-476.blu'

blue = Blueprinter.from_pose(pose)
blue[441:447] = 'NATAA'
del blue[447:476]
blue[477:478] = 'NATAA'
blue[479:483] = 'NATRO'
blue[484:493] = 'NATAA' # loop after helix

blue.set(bluprint_filename)

pyrosetta.rosetta.core.pose.remove_nonprotein_residues(pose)
pyrosetta.rosetta.basic.options.set_file_option('remodel:blueprint', bluprint_filename)
pyrosetta.rosetta.basic.options.set_string_option('remodel:generic_aa', 'G')
#pyrosetta.rosetta.basic.options.set_boolean_option('remodel:quick_and_dirty', True)
rm = pyrosetta.rosetta.protocols.forge.remodel.RemodelMover()
rm.apply(pose)
```
Correction. I think It worked actually.
```python
blue.correct(pose)
```

### Indel

The indel starts at a helix and ends with two turns of a helix.
Remodel was tried. But did not work.
As the C-terminal is unstructured, there is no point modelling it

```python
from pyrosetta_help.common_ops import get_last_res_in_chain
indel = pose.clone().split_by_chain()[1]
pyrosetta.rosetta.protocols.grafting.delete_region(indel, 967, get_last_res_in_chain(indel, 'A'))
print(len(indel.chain_sequence(1)))
blue = Blueprinter.from_pose(indel)
blue.rows[-1] = [966, 'I', 'H', 'NATAA']
for r in 'KAKKKIEQQ':
    blue.rows.append([0, 'X', 'H', f'PIKAA {r}'])
for r in 'GG': #FTFENKGVLSAFNFGTVPS':
    blue.rows.append([0, 'X', 'H', f'PIKAA {r}'])
```

However, it seems unable to do an extension in this version.
Making the last two residue wobble and remodelling the helix does not make the interveening un-remodelled positions swing.
The last part of the helix was therefore simply attached in PyMol to the helix before the truncation.
After all, it's an unbound helix and that would have happened.

