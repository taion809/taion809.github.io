---
date: 2014-11-27T13:57:00Z
title: "Preparing Integers With Doctrine"
description: "How do you add integers as prepared statements to a select?"
tags: [ "doctrine", "postgresql" ]
Section: blog
categories:
  - "Development"
slug: "prepare-int-select-statement"
---

I recently had an issue with binding an integer to a select statement with doctrine and postgresql.

<!--more-->

Given the following:

    <?php
        $sql = 'INSERT INTO tree (ancestor, descendant)
                SELECT cta.ancestor, :descendant
                FROM tree AS cta
                WHERE cta.descendant = :ancestor
                UNION ALL
                SELECT :descendant, :descendant';

        $statement = $connection->prepare($sql);
        $statement->bindValue("descendant", (int) $rootNode, 'integer');
        $statement->bindValue("ancestor", (int) $ancestor, 'integer');
        $statement->execute();
    ?>

I received the following error:

    /**
        PHP Fatal error:  Uncaught exception 'PDOException' with message 'SQLSTATE[42P08]: Ambiguous parameter: 7 ERROR:  inconsistent types deduced for parameter $1
        LINE 2:                 SELECT cta.ancestor, $1
                                                              ^
        DETAIL:  integer versus text' in /vagrant_data/vendor/doctrine/dbal/lib/Doctrine/DBAL/Driver/PDOStatement.php:91
    */

The solution was to explicitly cast it as an integer like so ::integer

And here is the revised code:

    <?php
        $sql = 'INSERT INTO tree (ancestor, descendant)
                SELECT cta.ancestor, :descendant::integer
                FROM tree AS cta
                WHERE cta.descendant = :ancestor
                UNION ALL
                SELECT :descendant, :descendant';

        $statement = $connection->prepare($sql);
        $statement->bindValue("descendant", (int) $rootNode, 'integer');
        $statement->bindValue("ancestor", (int) $ancestor, 'integer');
        $statement->execute();
    ?>

If there are errors or you have a suggestion send me a message @taio.
