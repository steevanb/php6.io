---
title: PersistentCollection et orphanRemoval
mainMenu: doctrine
---

# Annuler une demande de suppression d'entité

Depuis Doctrine ^2.5 (peut-être également les versions précédentes) : 
si on appelle `PersistentCollection::clear()` ou `PersistentCollection::removeElement()`,
et que la liaison a `orphanRemoval` : les éléments à supprimer seront enregistrés dans l'`UnitOfWork` comme étant à supprimer en base de données.

Jusque-là, tout va bien : c'est le comportement attendu.

Mais si on veut re-ajouter un élément indiqué comme à supprimer (pour annuler sa suppression par exemple) alors `PersistentCollection` n'annule pas la demande de suppression.

Résultat : l'élément supprimé puis re-ajouté est finalement supprimé en base alors qu'on voulait le conserver.

# Références

[Issue #6509](https://github.com/doctrine/doctrine2/issues/6509)
