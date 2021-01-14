# IPO8_analysis
An analysis of the RAN-IPO8 binding energies in different variants.

## Importin-8 description

Importins are protein involved in the nuclear import of protein and RNAs.
There are two different families, whose archetypes are importin alpha and beta.

* &alpha; binds to the nuclear localization signal on protein
* &alpha; and beta can form a dimer
* &beta; binds to the RAN GTPase, whose conformational change once activated triggers the exposure of a nuclear export signal.
* &beta; binds to the nuclear pore component Nup153
* Some homologues of &beta; can also bind to tRNA etc.
* Some &beta; homologues function solely for export of the &alpha; and are called exportins.
* Some &beta; homologues, such as IPO12 (transportin-1) can ferry cargo without the need of &alpha; and are called transportins.

Both the &beta; and the &alpha; homologues are composed of tetratrico peptide repeats (TPR, CL0020). These repeats have many variations, but are all distantly homologous.
In the case of &alpha; they are armadillo repeats (PF00514, a TPR fold) and there is also a &beta;-binding helical bundle (IBB, PF01749, also a TPR fold).
In the case of importin &beta; members the N-terminal is an importin &beta; N-terminal domain (PF03810, also a TPR fold),
in the middle, they are HEAT repeats (PF02985, also a TPR fold) and in the case of exportins (and IPO7 and IPO8) there is also a CSE domain (PF08506, also a TPR fold).

There are many protein, even in yeast. The complex there is also known as the karyopherin complex, resulting in its protein having Kap+number symbols.
Where in yeast there is only importin &alpha;, Kap60, SRP1, the &beta; homologues are more numerous.

* Kap95: importin &beta;
* Kap104: importin &beta;-2
* Kap121: importin &beta;-3
* Kap123: importin &beta;-4
* Kap114: importin &beta;-5
* Kap120: importin &beta; like
* Kap108: importin &beta; SMX1
* Kap119: NMD5 (Nonsense-mediated mRNA decay protein 5)
* CSE1 (not a kap protein): Importin &alpha; re-exporter (exportin)

Importin-8 is 50% identical to importin-7 and both are related to the yeast Nmd5p and Sxm1p. And more distantly to exportin.
In fact, they have the CSE domain.
The relationship between different importin &beta; family members is explored in [a paper](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0019308).
However, the key takehome message is that they have similar mechanisms, but different cargos.

There is not much evidence of importin &alpha; bindining, although it works with KPNB1 (importin &beta;) —but the KPNB1 interaction is teamwork not binding.
Whereas IPO7 (50% homologue of IPO8) seems to form a heterodimer with KPNB1, there is no evidence of any &beta; forming a homodimer or any other heterodimer with another &beta;.
So this latter link needs to be taken with a very strong pinch of salt.

The file [IPO8_yeast-PSSM_Scoremat.asn](IPO8_yeast-PSSM_Scoremat.asn) is a PSSM (_m_atrix), which can be used to find distant homologues with PSI-Blast in NCBI.

## Models

[Open with human RAN](IPO8-RAN.r.pdb): 1wa5 Swissmodel threaded model with human RAN from 6TVO placed where RAN is in 1WA5.

ITasser model used for spans resi 597-624 or resi 985-end, which had low confidence in the Swissmodel.
The Swissmodel, by virtue of it being based upon a RAN structure is better for the loop positions that contact RAN.

## Scripts

The base scripts can be found in [The base scripts can be found in [matteoferla/pyrosetta_scripts GitHub repo](The base scripts can be found in []https://github.com/matteoferla/pyrosetta_scripts)

