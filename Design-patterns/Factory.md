# Tutoriel : Utilisation des Factories en PHP

Dans ce tutoriel, nous allons explorer le concept des "factories" en PHP, un modèle de conception qui permet de créer des objets de manière flexible et modulaire. Les factories sont particulièrement utiles lorsque vous devez créer différents types d'objets avec des attributs et des comportements spécifiques.

## Pourquoi utiliser des Factories ?

Les factories offrent de nombreux avantages, notamment :

1. **Extensibilité** : Vous pouvez ajouter de nouveaux types d'objets sans modifier le code existant. Cela suit le principe d'ouverture/fermeture (Open/Closed Principle) du SOLID.

2. **Séparation des responsabilités** : Les factories se chargent de la création d'objets, ce qui évite d'avoir des constructeurs compliqués avec de nombreux paramètres. Cela centralise la logique de création.

3. **Facilité de tests** : Vous pouvez mock ou simuler une factory pour les tests unitaires, ce qui facilite les tests en isolant la création d'objets.

4. **Cohérence** : Les factories peuvent garantir que toutes les instances créées suivent certaines règles ou valeurs par défaut.

## Exemple de Code

Pour illustrer l'utilisation des factories, nous allons créer un système de gestion de tâches avec différents types de tâches : Tâche standard, Tâche avec un délai et Tâche de révision. Chacun de ces types de tâches a des attributs spécifiques.

Voici un exemple de code :
```php
<?php
// Classe pour représenter un utilisateur
class User {
    private string $username;

    public function __construct(string $username) {
        $this->username = $username;
    }

    public function getUsername(): string {
        return $this->username;
    }
}

// Classe abstraite pour représenter toutes les tâches
abstract class Task {
    protected string $title;
    protected string $description;
    protected string $status;
    protected string $dueDate;
    protected array $assignedUsers = [];

    public function __construct(string $title, string $description, string $status, string $dueDate) {
        $this->title = $title;
        $this->description = $description;
        $this->status = $status;
        $this->dueDate = $dueDate;
    }

    public function assignUser(User $user): void {
        $this->assignedUsers[] = $user;
    }

    abstract public function formatAsString(): string;

    protected function formatTaskDetails(): string {
        $taskString = "Title: {$this->title}\n";
        $taskString .= "Description: {$this->description}\n";
        $taskString .= "Status: {$this->status}\n";
        $taskString .= "Due Date: {$this->dueDate}\n";

        if (!empty($this->assignedUsers)) {
            $taskString .= "Assigned Users:\n";
            foreach ($this->assignedUsers as $user) {
                $taskString .= "- {$user->getUsername()}\n";
            }
        }

        return $taskString;
    }
}

// Tâche standard
class StandardTask extends Task {
    public function formatAsString(): string {
        return $this->formatTaskDetails();
    }
}

// Tâche avec délai critique
class CriticalTask extends Task {
    private string $priority;

    public function __construct(string $title, string $description, string $status, string $dueDate, string $priority) {
        parent::__construct($title, $description, $status, $dueDate);
        $this->priority = $priority;
    }

    public function formatAsString(): string {
        $taskString = $this->formatTaskDetails();
        $taskString .= "Priority: {$this->priority}\n";
        return $taskString;
    }
}

// Tâche de révision
class ReviewTask extends Task {
    private string $author;
    private string $comment;

    public function __construct(string $title, string $description, string $status, string $dueDate, string $author, string $comment) {
        parent::__construct($title, $description, $status, $dueDate);
        $this->author = $author;
        $this->comment = $comment;
    }

    public function formatAsString(): string {
        $taskString = $this->formatTaskDetails();
        $taskString .= "Author: {$this->author}\n";
        $taskString .= "Comment: {$this->comment}\n";
        return $taskString;
    }
}

// Factory pour créer des tâches
class TaskFactory {
    public static function createStandardTask(string $title, string $description, string $status, string $dueDate): StandardTask {
        return new StandardTask($title, $description, $status, $dueDate);
    }

    public static function createCriticalTask(string $title, string $description, string $status, string $dueDate, string $priority): CriticalTask {
        return new CriticalTask($title, $description, $status, $dueDate, $priority);
    }

    public static function createReviewTask(string $title, string $description, string $status, string $dueDate, string $author, string $comment): ReviewTask {
        return new ReviewTask($title, $description, $status, $dueDate, $author, $comment);
    }
}

$standardTask = TaskFactory::createStandardTask("Standard Task", "Description for Standard Task", "Incomplete", "2023-12-31");
$criticalTask = TaskFactory::createCriticalTask("Critical Task", "Description for Critical Task", "Complete", "2023-10-15", "High");
$reviewTask = TaskFactory::createReviewTask("Review Task", "Description for Review Task", "In Progress", "2023-12-31", "John Doe", "Consider changing the color of the header.");

$user1 = new User("UserA");
$standardTask->assignUser($user1);

$user2 = new User("UserB");
$user3 = new User("UserC");
$criticalTask->assignUser($user2);
$criticalTask->assignUser($user3);

$reviewTask->assignUser($user2);


echo "Standard Task:\n";
echo $standardTask->formatAsString();

echo "\n\nCritical Task:\n";
echo $criticalTask->formatAsString();

echo "\n\nReview Task:\n";
echo $reviewTask->formatAsString();

```
Dans cet exemple, nous avons créé trois types de tâches (Standard, Critique, Révision) et utilisé des factories pour les créer. Cette approche présente plusieurs avantages essentiels : 
1. **Modularité** : Grâce à l'utilisation de factories, chaque type de tâche est encapsulé dans sa propre classe, ce qui favorise une conception modulaire. Vous pouvez facilement ajouter de nouveaux types de tâches en créant de nouvelles classes de tâches et en étendant la factory correspondante, sans perturber le fonctionnement des autres parties de votre code. Cela signifie que votre code reste structuré, facile à gérer et à étendre, ce qui est particulièrement important dans les projets de grande envergure.
2. **Flexibilité** : Les factories vous permettent de créer des objets avec des configurations spécifiques en utilisant des paramètres adaptés à chaque type de tâche. Par exemple, une tâche critique peut nécessiter un paramètre supplémentaire tel que la priorité, tandis qu'une tâche de révision peut nécessiter l'indication de l'auteur et un commentaire. Cette flexibilité vous permet de créer des objets avec les attributs appropriés pour chaque situation, ce qui est particulièrement utile dans un environnement où les besoins peuvent évoluer.
3. **Maintenabilité** : En centralisant la logique de création des objets dans des factories dédiées, votre code devient plus facile à entretenir. Si vous devez apporter des modifications aux paramètres de création ou à la logique de création elle-même, vous n'avez qu'à le faire dans la factory correspondante, sans affecter le reste de votre application. Cela réduit les risques d'erreurs et facilite la maintenance à long terme de votre code.

## Conclusion
Les factories sont un outil puissant pour la création d'objets en PHP. Elles permettent de rendre votre code plus extensible, plus lisible et plus facile à tester. En les utilisant correctement, vous pouvez simplifier la création d'objets complexes tout en conservant une structure propre et modulaire dans votre code.