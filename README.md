import streamlit as st
import re

# Configuration de la page
st.set_page_config(
    page_title="Calculateur de Coûts d'Impression - Maison d'Édition",
    page_icon="📚",
    layout="centered")

# Titre et introduction
st.title("Calculateur de Coûts d'Impression")
st.subheader("Estimez le coût d'impression de vos livres")
st.markdown("---")


# Fonction pour valider que l'entrée est un nombre positif
def est_nombre_positif(valeur):
    if not valeur:
        return False
    # Remplacer la virgule par un point pour gérer le format français
    valeur = valeur.replace(',', '.')
    # Vérifier que c'est un nombre positif
    try:
        nombre = float(valeur)
        return nombre > 0
    except ValueError:
        return False


# Fonction de calcul du coût
def calculer_cout(collector,
                  nb_livre,
                  nb_page_manuscrite=0,
                  nb_page_illustration=0,
                  nb_page=0):
    montant = 0.0

    if collector == "oui":
        montant = nb_page_illustration * 0.05 + nb_page_manuscrite * 0.04
    else:  # collector == "non"
        if nb_page > 850:
            montant = 5.90
        elif nb_page > 600:
            montant = 4.90
        elif nb_page > 450:
            montant = 3.90
        elif nb_page > 200:
            montant = 2.90
        else:
            montant = 1.90

    montant = nb_livre * montant
    return montant


# Type de livre (en dehors du formulaire pour une mise à jour immédiate)
collector = st.radio("Le livre est-il un collector ?",
                     options=["non", "oui"],
                     horizontal=True)

# Formulaire de calcul
with st.form("formulaire_cout"):
    st.subheader("Détails du Projet d'Impression")

    # Nombre de livres à imprimer
    nb_livre_str = st.text_input("Combien de livres souhaitez-vous imprimer ?")

    # Champs conditionnels basés sur le type de livre
    if collector == "oui":
        nb_page_manuscrite_str = st.text_input(
            "Combien de pages manuscrites ?")
        nb_page_illustration_str = st.text_input(
            "Combien de pages d'illustration ?")
        nb_page_str = None
    else:
        nb_page_str = st.text_input("Combien de pages à imprimer ?")
        nb_page_manuscrite_str = None
        nb_page_illustration_str = None

    # Bouton de calcul
    btn_calculer = st.form_submit_button("Calculer le coût")

# Traitement après soumission
if btn_calculer:
    erreurs = []

    # Validation du nombre de livres
    if not est_nombre_positif(nb_livre_str):
        erreurs.append("Le nombre de livres doit être un nombre positif.")

    # Validation des pages selon le type de livre
    if collector == "oui":
        if not est_nombre_positif(nb_page_manuscrite_str):
            erreurs.append(
                "Le nombre de pages manuscrites doit être un nombre positif.")
        if not est_nombre_positif(nb_page_illustration_str):
            erreurs.append(
                "Le nombre de pages d'illustration doit être un nombre positif."
            )
    else:
        if not est_nombre_positif(nb_page_str):
            erreurs.append("Le nombre de pages doit être un nombre positif.")

    # Affichage des erreurs s'il y en a
    if erreurs:
        for erreur in erreurs:
            st.error(erreur)
    else:
        # Conversion des entrées en nombres
        nb_livre = float(nb_livre_str.replace(',', '.'))

        if collector == "oui":
            nb_page_manuscrite = float(nb_page_manuscrite_str.replace(
                ',', '.'))
            nb_page_illustration = float(
                nb_page_illustration_str.replace(',', '.'))
            nb_page = 0
        else:
            nb_page = float(nb_page_str.replace(',', '.'))
            nb_page_manuscrite = 0
            nb_page_illustration = 0

        # Calcul du coût
        cout_total = calculer_cout(collector, nb_livre, nb_page_manuscrite,
                                   nb_page_illustration, nb_page)

        # Affichage du résultat
        st.markdown("---")
        st.subheader("Résultat")

        # Carte de résultat
        st.success(f"Montant total : {cout_total:.2f} €")

        # Détails du calcul
        with st.expander("Détails du calcul"):
            if collector == "oui":
                st.write(
                    f"- Coût par livre: {(nb_page_manuscrite * 0.04 + nb_page_illustration * 0.05):.2f} €"
                )
                st.write(
                    f"  • Pages manuscrites: {nb_page_manuscrite} × 0.04 € = {nb_page_manuscrite * 0.04:.2f} €"
                )
                st.write(
                    f"  • Pages d'illustration: {nb_page_illustration} × 0.05 € = {nb_page_illustration * 0.05:.2f} €"
                )
            else:
                prix_unitaire = 0
                if nb_page > 850:
                    prix_unitaire = 5.90
                elif nb_page > 600:
                    prix_unitaire = 4.90
                elif nb_page > 450:
                    prix_unitaire = 3.90
                elif nb_page > 200:
                    prix_unitaire = 2.90
                else:
                    prix_unitaire = 1.90

                st.write(f"- Nombre de pages: {nb_page}")
                st.write(f"- Tranche de prix: {prix_unitaire} € par livre")

            st.write(f"- Nombre de livres: {nb_livre}")
            st.write(f"- Coût total: {cout_total:.2f} €")

# Informations supplémentaires
st.markdown("---")
with st.expander("Grille tarifaire"):
    st.subheader("Édition standard (non collector)")
    st.write("- Jusqu'à 200 pages: 1,90 € par livre")
    st.write("- De 201 à 450 pages: 2,90 € par livre")
    st.write("- De 451 à 600 pages: 3,90 € par livre")
    st.write("- De 601 à 850 pages: 4,90 € par livre")
    st.write("- Plus de 850 pages: 5,90 € par livre")

    st.subheader("Édition collector")
    st.write("- Pages manuscrites: 0,04 € par page")
    st.write("- Pages d'illustration: 0,05 € par page")
