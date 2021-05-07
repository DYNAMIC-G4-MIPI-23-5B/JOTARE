# Notre code Python
<br/>

import matplotlib.pyplot as plt
import numpy as np
In [14]:
class individu():
    def __init__ (self,age,esperance,mutation, sexe):
        self.a=age   #int
        self.e=esperance   #int
        self.m=mutation   #int entre 0 (pour aucun allèle muté) et 2 (pour un allèle muté)
        self.s=sexe  #0 pour homme, 1 pour femme
        
def esperence(moy, maxi):
    """
    moy:int est la moyenne d'age de la population et maxi:int est l'age maximum de mort des individus    
    """
    return random.choice([e for e in range (0,maxi)]+[e for e in range (moy//2,(maxi+moy//2))]+ [e for e in range (2//3*moy,(2*moy+maxi)//3)]+[moy+1,moy])

def naissance (esperence, mutation, sexe):
    return individu(0, esperance, mutation, sexe)
In [15]:
def genere_indiv(maxi, proba_mut):
    """Appelé par la fonction init_popu
    Renvoie un individu dont les paramètres sont définis aléatoirement (l'espérence est entre 0 et max et il a proba_mut*2 chances de posséder la mutation)"""
    
    #le nombre d'allèles mutés que possède l'individu
    mutation=0
    random.seed(0,1)
    if random.random()>=proba_mut:
        mutation+=1
    if random.random()>=proba_mut:
        mutation+=1
    
    random.seed(0,maxi)
    return individu (0, math.floor(random.random()),mutation, random.choice([0,1]))


def init_popu(nb_indiv,taille_cases, maxi, proba_mut):
    """
    nb_indiv: int : nombre d'individus présents dans la population au départ, NB_INDIV DOIT ETRE UN MULTIPLE DE TAILLE_CASES
    taille_cases: int : plus le nombre de cases est grand, plus l'interaction entre individus de la population est forte
    Retourne un array représentant la populatio initiale
    """
    pop=np.array([genere_indiv(maxi,proba_mut) for e in range(0,nb_indiv)])
    return np.reshape(pop, (-1,nb_indiv))

init_popu(10,2,10, 2)
#genere_indiv(10, 40)
Out[15]:
array([[<__main__.individu object at 0x00000259C1D7B670>,
        <__main__.individu object at 0x00000259C1D7B6A0>,
        <__main__.individu object at 0x00000259C1D7B790>,
        <__main__.individu object at 0x00000259C1D7B460>,
        <__main__.individu object at 0x00000259C1D7B7C0>,
        <__main__.individu object at 0x00000259C1D7B880>,
        <__main__.individu object at 0x00000259C1D7B8E0>,
        <__main__.individu object at 0x00000259C1D7B640>,
        <__main__.individu object at 0x00000259C1D7B970>,
        <__main__.individu object at 0x00000259C1D7BA00>]], dtype=object)
In [ ]:
def fecondation(male, femelle, dominant, avantage, moy, maxi)->naissance:
   """
    femelle et male sont de type individu
    moy et maxi sont des int 
    dominant indique si l'allèle muté est dominant et l'espérence moyenne de l'individu est augmentée de avantage s'il s'exprime
    Renvoie un individu issu de la fécondation entre le male et la femelle entrés en paramètre
    """
    #initialise mutation: le nombre d'allèle que possèdera l'enfant en fonction des allèles de ses parent
    mut = male.m + femelle.m
    if mut==4:
        mutation=2
    if mut==3:
        mutation=random.choice([1,2])
    if mut==2:
        if male.m==2 or femelle.m==2:
            mutation=1
        else:
            mutation=random.choice([2,1,1,0])
    if mut==1:
        mutation=random.choice([0,0,0,1])
    else:
        mutation=0
    
    #appelle la fonction naissance, avec une espérence modifiée si l'individu possède un allèle muté qui s'exprime
    if dominant:
        if mutation:
            return naissance(0, esperance(moy+avantge,maxi), mutation, random.choice([0,1]))
        return naissance(0, esperance(moy,maxi), mutation, random.choice([0,1]))
    if mutation==2:
        return naissance(0, esperance(moy+avantge,maxi), mutation, random.choice([0,1]))
    
    return naissance(0, esperance(moy,maxi), mutation, random.choice([0,1]))
In [16]:
def annee_groupe(groupe, fertil_inf, fertil_sup, dominant, avantage, proba_repro,naissances):
    """Appellée par la fonction annee, sert a mettre a jour l'arrray pop et la liste naissances"""
    
    #listes des males et femelles fertils du groupe
    male=[]
    femelle=[]
    
    # pour tous les individus: augmente l'age, diminue l'esperance, le supprime si l'esperance est nulle, l'ajoute à la liste male ou femelle s'il est fertil
    for e in groupe:
        e.e-=1   #modifie esperance
        if e.e<=0:   #supprime les morts
            e=0
            continue
        e.a+=1
        if e.a>fertil_inf and e.a>fertil_sup:    #ajoute a la liste male ou femelle
            if e.s==0:
                male.append(e)
            else:
                femelle.append(e)
    
    random.seed(0,1)
    for m in male:   #a chaque fois, tire un nombre aleatoire entre 0 & 1, s'il est supérieur à proba_repro, déclenche la fonction fecondite
        for f in femelle:
            if random.random()>=proba:
                naissances.append(fecondidte(e,f,moy,maxi))
In [17]:
def annee(pop, taille_cases,   fertil_inf, fertil_sup,  proba_repro,  dominant,avantage,  moy,maxi):
    """ SIMULATION D UNE ANNEE DANS LA POPULATION
    
            pop:array
            taille_cases est le nombre d'individus par listes
            fertil_inf et fertil_sup sont les bornes entre lesquelles un individu est fertil
            
            proba_repro est la probabilité que un male et une femelle qui se rencontrent fassent un enfant
            
            dominant indique si le gène est dominant
            avantage indique le nombre d'années en moyenne que l'individu muté vivra en plus
                Précondition : avantage+moy < max
                
            moy est l'esperance de vie moyenne de la population
            max est l'age maximum pour un inidividu de la population
    """
    
    #liste des nouveaux nés (va être remplie par l'appel de la fonction annee_groupe)
    naissances=[]
    
    #appel la fonction annee_goupe sur chacune des listes de l'array pop
    for e in pop:
        annee_groupe(e, fertil_inf, fertil_sup, proba_repro,naissances,moy,maxi)
    
    #ajoute les nouveaux nés dans pop (pas tous au même endroit)
    no_listes=[e for e in range (0,len(pop))]
    for e in naissances:
        np.append(pop, e, axis=random.choice(no_listes))
    
    return np.reshape(pop,(-1,taille_cases))
In [18]:
def evolution(nb_annees,  pop,  nb_inidv,taille_cases,proba_mut,   fertil_inf, fertil_sup,  proba_repro,  dominant,avantage,  moy,maxi):
    """     nb_annees est le nombre d'années qui vont s'écouler dans la simulation
    
            pop:array
            
            nb_indiv est le nombre d'individus au départ
            taille_case est la taille de chaque liste de l'array, plus elle est grande, plus les interaction sont fortes
            proba_mut:float entre 0 et 1 est la proportion d'individus mutés au départ
            
            fertil_inf et fertil_sup sont les bornes entre lesquelles un individu est fertil
            
            proba_repro est la probabilité que un male et une femelle qui se rencontrent fassent un enfant
            
            dominant indique si le gène est dominant
            avantage indique le nombre d'années en moyenne que l'individu muté vivra en plus
                Précondition : avantage+moy < max
                
            moy est l'esperance de vie moyenne de la population
            max est l'age maximum pour un inidividu de la population
        Fonction finale qui retourne la population après nb_annees
    """
    
    #appelle la fonction annee nb_annees fois
    while (nb_annees):
        annee (pop, taille_cases,   fertil_inf, fertil_sup,  proba_repro,  dominant,avantage,  moy,maxi)
        nb_annees-=1
    
    return pop
<a href="notrenotebook-checkpoint.ipynb"> Le document notebook est là </a>
<br/>
<br/>
<br/>
<br/>
<br/>
<a href="index.html"> Retour à l'accueil </a>
