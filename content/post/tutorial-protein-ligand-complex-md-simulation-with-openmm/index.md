---
title: "Tutorial: Protein-Ligand Complex MD Simulation with OpenMM"
date: 2022-06-17T09:01:17.966Z
summary: OpenMM toolkit을 이용한 Protein-Ligand Complex MD Simulation 맛보기
draft: false
featured: false
image:
  filename: featured
  focal_point: Smart
  preview_only: false
---
간단하게 python으로 molecular dymanics simulation을 돌릴 수 있도록 여러 기능들을 제공하는 [OpenMM toolkit](https://openmm.org)을 사용하여, 단백질-리간드 복합체에 대한 MD tutorial을 정리해보았습니다. [tdudgeon님의 GitHub](https://github.com/tdudgeon/simple-simulate-complex) 의 내용을 많이 참고하였습니다.

1. **프로그램 설치**
    
    conda를 이용하여 다음과 같이 설치를 합니다(만, 저는 system conflict 에러가 계속 나서 [mamba](https://mamba.readthedocs.io/en/latest/#)를 통해 설치하였습니다).
    
    ```jsx
    conda create -n openmm_test python=3.7.12
    conda activate openmm_test
    conda install -c conda-forge openmm 
    conda install -c conda-forge openff-toolkit
    conda install -c conda-forge openmmforcefields
    ```
    
    설치 후 파이썬 스크립트 상에서 아래 모듈들이 정상적으로 import 되는지 확인해야합니다.
    
    ```jsx
    from openff.toolkit.topology import Molecule
    from openmmforcefields.generators import SystemGenerator
    from simtk import unit
    from pdbfixer import PDBFixer
    from openmm import app, Platform, LangevinIntegrator
    from openmm.app import PDBFile, Simulation, Modeller, PDBReporter, DCDReporter, StateDataReporter
    ```
    
2. **Ligand 및 Receptor의 Input file 준비**
    
    이제 MD simulation에 input으로 사용할 단백질과 리간드의 파일을 준비해야합니다. 
    
    - 단백질 파일의 경우 물을 포함한 기타 분자들을 없애고 빠진 수소 원자를 붙여주어야 합니다.
        
        ```jsx
        fixer = PDBFixer(filename="protein_ori.pdb")
        fixer.findMissingResidues()
        fixer.findMissingAtoms()
        fixer.findNonstandardResidues()
        
        print('Residues:', fixer.missingResidues)
        print('Atoms:', fixer.missingAtoms)
        print('Terminals:', fixer.missingTerminals)
        print('Non-standard:', fixer.nonstandardResidues)
        
        fixer.addMissingAtoms()
        fixer.addMssingHydrogens(7.4)
        
        fixer.removeHeterogens(False)
        
        with open("protein_cleaned.pdb", 'w') as w:
        		PDBFile.writeFile(fixer.topology, fixer.positions, file=w, keepIds=True)
        ```
        
    - 리간드 파일의 경우 수소를 달아주어야합니다. `modeller.addHydrogens()` 가 리간드에 대해서는 잘 적용되지 않아, `rdkit.Chem.AddHs(mol, addCoords=True)` 을 사용하였습니다.
        
        ```jsx
        from rdkit import Chem
        
        mol = Chem.SDMolSupplier("ligand_ori.sdf")[0]
        mol_addh = Chem.AddHs(mol, addCoords=True)
        Chem.AssignAtomChiralTagsFromStructure(mol_addh)
        
        writer = Chem.SDWriter("ligand_cleaned.sdf")
        writer.write(mol_addh)
        writer.close()
        ```
        

1. **초기 파라미터 설정**
    
    MD simulation을 위한 파라미터 및 파일 이름들을 아래와 같이 설정해주었습니다. 
    
    - `temperature`: Simulation을 진행할 시스템의 온도 (단위: K)
    - `equilibration_steps`: Equilibrium을 진행할 step의 횟수 (1 step = 2 fs)
    - `reporting_interval`: Trajectory를 몇 step 마다 저장할지
    - `num_md_steps`: MD simulation을 진행할 step의 횟수 (500,000 steps = 1 ns)
    
    ```jsx
    temperature = 300 * unit.kelvin
    equilibration_steps = 200
    reporting_interval = 500
    num_md_steps = 500000
    
    protein_pdb = "protein_cleaned.pdb"
    ligand_sdf = "ligand_cleaned.sdf"
    output_complex_pdb = "output_complex.pdb"
    output_traj_dcd = "output_traj.dcd"
    
    forcefield_kwargs = {
    				"constraints": app.HBonds,
    				"rigidWater": True,
    				"removeCMMotion": False,
    				"hydrogenMass": 4 * unit.amu
    }
    ```
    
    추가로, GPU 사용을 위하여 아래와 같이 CUDA 설정을 해줍니다.
    
    ```jsx
    speed = 0
    for i in range(Platform.getNumPlatforms()):
        p = Platform.getPlatform(i)
        if p.getSpeed() > speed:
            platform = p
            speed = p.getSpeed()
    
    if platform.getName() == 'CUDA':
        platform.setPropertyDefaultValue('Precision', 'mixed')
    ```
    
2. **단백질-리간드 복합체 시스템 생성**
    
    ```jsx
    ligand_mol = Molecule(Chem.SDMolSupplier(ligand_sdf)[0])
    protein_mol = PDBFile(protein_pdb)
    
    modeller = Modeller(protein_mol.topology, protein_mol.positions)
    modeller.add(ligand_mol.to_topology().to_openmm(), ligand_mol.conformers[0])
    
    with open(output_complex_pdb, 'w') as w:
    		PDBFile.writeFile(modeller.topology, modeller.positions, w)
    
    system_generator = SystemGenerator(
    				forcefields=['amber/ff14SB.xml'],
    				small_molecule_forcefield='gaff-2.11',
    				forcefield_kwargs=forcefield_kwargs
    )
    system = system_generator.create_system(
    				modeller.topology, 
    				molecules=ligand_mol
    )
    ```
    
3. **Energy Minimization**
    
    ```jsx
    integrator = LangevinIntegrator(
    				temperature=temperature, 
    				frictionCoeff=1 / unit.picosecond, 
    				stepSize=0.002 * unit.picoseconds
    )
    
    simulation = Simulation(
    				topology=modeller.topology, 
    				system=system, 
    				integrator=integrator, 
    				platform=platform
    )
    simulation.context.setPositions(modeller.positions)
    simulation.minimizeEnergy()
    ```
    
4. **Equilibration**
    
    ```jsx
    simulation.context.setVelocitiesToTemperature(temperature)
    simulation.step(equilibration_steps)
    ```
    
5. **MD Simulation**
    
    ```jsx
    simulation.reporters.append(DCDReporter(output_traj_dcd, reporting_interval, enforcePeriodicBox=False))
    simulation.reporters.append(StateDataReporter(sys.stdout, reporting_interval * 5, step=True, potentialEnergy=True, temperature=True))
    simulation.step(num_md_steps)
    ```
    
6. **Analyze RMSD**
    
    ```jsx
    import mdtraj as md
    import matplotlib.pyplot as plt
    import pandas as pd
    import seaborn as sns
    
    t = md.load(
    				output_traj_dcd,
    				top=output_complex_pdb
    )
    
    atoms = t.topology.select("chainid 1")
    rmsds_lig = md.rmsd(t, t, frame=0, atom_indices=atoms, parallel=True, precentered=False)
    atoms = t.topology.select("chainid 0 and backbone")
    rmsds_bck = md.rmsd(t, t, frame=0, atom_indices=atoms, parallel=True, precentered=False)
    
    df = pd.DataFrame({"time": t.time, "lig": rmsds_lig, "bck": rmsds_bck})
    
    sns.lineplot(df, x="time", y="lig")
    sns.lineplot(df, x="time", y="bck")  
    
    plt.show()
    ```
    
7. **Visualize Trajectory**
    
    `pymol` 을 연 후, cmd 창에서 아래와 같이 입력하여 MD Simulation 결과를 visualize 할 수 있습니다.
    
    ```jsx
    load $output_complex_pdb
    load_traj $output_traj_dcd
    ```
    

---

감사합니다. 👍

[1] Eastman, Peter, et al. "OpenMM 7: Rapid development of high performance algorithms for molecular dynamics." *PLoS computational biology* 13.7 (2017): e1005659.

[2] https://github.com/tdudgeon/simple-simulate-complex

[3] [http://docs.openmm.org/7.1.0/api-python/index.html](http://docs.openmm.org/7.1.0/api-python/index.html)