# Introduction #

To run MOE, run the Moe binary. The first argument should be the "directive" to invoke. Most directives take in one or more command options. Running a MOE directive with the "--help" option will list the command options needed for that directive.

# The Directives #

## hello ##
The simplest directive.
```
$ Moe hello
Hello MOE.
```

## check\_config ##
The check\_config directive checks that our configuration is valid.
```
$ Moe check_config --config_file moe_config.txt
Reading config file from moe_config.txt... Done
```

## highest\_revision ##
MOE often needs to know the highest revision in a source control repository:
```
$ Moe highest_revision --config_file moe_config.txt --repository 'googlecode{3}'
Reading config file from moe_config.txt... Done
Highest revision in repository "googlecode": 3

$ Moe highest_revision --config_file moe_config.txt --repository 'googlecode'
Reading config file from moe_config.txt... Done
Highest revision in repository "googlecode": 5
```
Note that the --repository flag takes a [Revision Expression](ExpressionLanguages#Revision_Expressions.md) stating the repository and the max revision to look from. If no max revision is specified, the default is HEAD.

## create\_codebase ##
Gets the latest version of a repository as in another repository. For instance, if we want to get the latest version of our code as in the repository "googlecode", we run:
```
$ Moe create_codebase --config_file moe_config.txt --codebase 'googlecode'
Reading config file from moe_config.txt... Done
Evaluating Codebase "googlecode"...
    Creating from "googlecode"... /tmp/svn_export_googlecode_5_8020984248422681827
DONE: Evaluating Codebase "googlecode": /tmp/svn_export_googlecode_5_8020984248422681827
Codebase "googlecode" created at /tmp/svn_export_googlecode_5_8020984248422681827
```
The --codebase flag takes a [Codebase Expression](ExpressionLanguages#Codebase_Expressions.md).

## change ##
Creates a (pending) change.
The following will create the codebase (just as if we ran create\_codebase) from the internal repository. It will also create a Writer from the googlecode repository:
```
$ Moe change --config_file moe_config.txt --codebase 'internal' --destination 'googlecode'
Reading config file from moe_config.txt... Done
Creating a change in "googlecode" with contents "internal"
Evaluating Codebase "internal"...
    Creating from "internal": /tmp/expanded_tar_86346705806076214
DONE: Evaluating Codebase "internal": /tmp/expanded_tar_86346705806076214
Creating Writer "googlecode"... /tmp/svn_writer_7_2455561459661896424
Putting files from Codebase into Writer... Done
Created Draft Revisions: /tmp/svn_writer_7_2455561459661896424
```
The [Expression Language](ExpressionLanguages.md) page describes in more detail how to specify additional options, as --codebase and --destination both take Codebase Expressions.
In the example above, a useful option is to be able to specify a translator. To do this to translate the codebase created from the internal repository from an internal project space to a public project space, use
```
$ Moe change --config_file moe_config.txt --codebase 'internal>public' --destination 'googlecode'
Reading config file from moe_config.txt... Done
Creating a change in "googlecode" with contents "internal>public"
Evaluating Codebase "internal>public"...
    Creating from "internal"... /tmp/expanded_tar_7371717651236131767
    Translating /tmp/expanded_tar_7371717651236131767 from project space "internal" to "public"... 
        Editing /tmp/expanded_tar_7371717651236131767 with editor "scrub_step"... /tmp/expanded_tar_3148939818423891966
        Editing /tmp/expanded_tar_3148939818423891966 with editor "shell step shell_step"... /tmp/shell_run_3190101037665555028
        Editing /tmp/shell_run_3190101037665555028 with editor "rename step rename_step"... /tmp/rename_run_8812192545396128575
    DONE: Translating /tmp/expanded_tar_7371717651236131767 from project space "internal" to "public": /tmp/rename_run_8812192545396128575
DONE: Evaluating Codebase "internal>public": /tmp/rename_run_8812192545396128575
Creating Writer "googlecode"... /tmp/svn_writer_5_1004792749160326091
Putting files from Codebase into Writer... Done
Created Draft Revision: /tmp/svn_writer_5_1004792749160326091
```

## find\_equivalence ##
Equivalences between revisions are stored in the MOE database and can be accessed using the find\_equivalence directive.

The --revision flag takes a [Revision Expression](ExpressionLanguages#Revision_Expressions.md) describing the revision to find equivalences to in the repository given to the --in\_repository flag.
```
$ Moe find_equivalence --config_file moe_config.txt --db moe_db.txt --revision 'googlecode{2}' --in_repository 'internal'
Reading config file from moe_config.txt... Done
"googlecode{2}" == "internal{12345678}"
```

## revisions\_since\_equivalence ##
This finds all of the revisions in a repository since the last equivalence between that repository and another repository. The --from\_repository flag takes a [Revision Expression](ExpressionLanguages#Revision_Expressions.md), allowing optional specification of the highest revision in that repository to consider. Note that to\_repository is just the name of the other repository.
```
$ Moe revisions_since_equivalence --config_file moe_config.txt --db moe_db.txt --from_repository 'internal{12345678}' --to_repository 'googlecode'
Reading config file from moe_config.txt... Done
Revisions found: internal{12345678}, internal{12345677}, internal{12345676}
```

## diff\_codebases ##
Prints the diff between two codebases, which are specified by [Codebase Expressions](ExpressionLanguages#Codebase_Expressions.md). For example, you could diff '`googlecode(revision=5)`' with '`googlecode(revision=6)`' to see what changes [revision 6](https://code.google.com/p/moe-java/source/detail?r=6) made to 5. Or you could diff "`internal>public`" with "`googlecode`" to see what differences the translated internal repository has with the head revision of the googlecode repository. This directive generates both codebases and then diffs them.
```
$ Moe diff_codebases --config_file moe_config.txt --codebase1 'internal>public' --codebase2 'googlecode'
Reading config file from moe_config.txt... Done
Evaluating Codebase "internal>public"... 
  Creating from "internal"... /tmp/expanded_tar_899180868266910054
  Translating /tmp/expanded_tar_899180868266910054 from project space "internal" to "public"... 
    Editing /tmp/expanded_tar_899180868266910054 with editor "scrub_step"... /tmp/expanded_tar_1226026764668686766
    Editing /tmp/expanded_tar_1226026764668686766 with editor "shell step shell_step"... /tmp/shell_run_4330577915735429998
    Editing /tmp/shell_run_4330577915735429998 with editor "rename step rename_step"... /tmp/rename_run_1419770579287133662
  DONE: Translating /tmp/expanded_tar_899180868266910054 from project space "internal" to "public": /tmp/rename_run_1419770579287133662
DONE: Evaluating Codebase "internal>public": /tmp/rename_run_1419770579287133662
Evaluating Codebase "googlecode"... 
  Creating from "googlecode"... /tmp/svn_export_googlecode_5_6692355302117543190
DONE: Evaluating Codebase "googlecode": /tmp/svn_export_googlecode_5_6692355302117543190
Codebases "internal>public" and "googlecode" are identical
```

## determine\_metadata ##
This directive combines the metadata (i.e. author, date, description, and parents) of the revisions specified by the [Revision Expression](ExpressionLanguages#Revision_Expressions.md). The migration directive makes use of this directive when combining multiple changes in one repository into one change in another repository. Generally, users won't explicitly use this directive.
```
$ Moe determine_metadata --config_file moe_config.txt --revisions 'googlecode{3,4}'
Reading config file from moe_config.txt... Done
id: 3, 4
author: user1@google.com, user2@google.com
date: 2011-06-27T23:43:00.581857Z, 2011-07-13T21:48:36.665584Z
description: A fourth commit.
     Change on 2011-06-27T23:43:00.581857Z by user1@google.com
-------------
A third commit.
     Change on 2011-07-13T21:48:36.665584Z by user2@google.com
General MOE update. Includes beginnings of Mercurial support.
parents: [googlecode{2}, googlecode{3}]
```

## one\_migration ##
This directive performs a single migration. Each migration corresponds to one change command. Like the change command, one\_migration is directional, and makes a draft revision. It differs from change in that it gets (and combines) the metadata from the revisions listed in --revisions\_to\_migrate to use when preparing the change for submission. This directive generally should not be called explicitly; use change to generate a draft revision without metadata, and the migrate directive (next) for draft revisions including metadata.
```
$ Moe one_migration --config_file moe_config.txt --db moe_db.txt --from_revision 'internal{22651520}' --to_revision 'googlecode' --revisions_to_migrate 'internal{22651520,22648237}'
Reading config file from moe_config.txt... Done
Evaluating Codebase "internal(revision=22651520)>public"... 
    Creating from "internal(revision=22651520)"... /tmp/expanded_tar_5873094244714001923
    Translating /tmp/expanded_tar_5873094244714001923 from project space "internal" to "public"... 
        Editing /tmp/expanded_tar_5873094244714001923 with editor "scrub_step"... /tmp/expanded_tar_1816569743026066832
        Editing /tmp/expanded_tar_1816569743026066832 with editor "shell step shell_step"... /tmp/shell_run_7144616482586614871
        Editing /tmp/shell_run_7144616482586614871 with editor "rename step rename_step"... /tmp/rename_run_3166784235682369951
    DONE: Translating /tmp/expanded_tar_5873094244714001923 from project space "internal" to "public": /tmp/rename_run_3166784235682369951
DONE: Evaluating Codebase "internal(revision=22651520)>public": /tmp/rename_run_3166784235682369951
Creating Writer "googlecode"... /tmp/svn_writer_5_1251170501963695488
Creating a change in "googlecode" with revisions "internal{22651520,22648237}"
Putting files from Codebase into Writer... Done
Created Draft Revision: /tmp/svn_writer_5_1251170501963695488
```

## migrate ##
The migrate directive performs a migration as specified by the migration section of the [MOE config](Config.md). It determines which revisions are new since equivalence, and passes those to the one\_migration directive, after performing the specified metadata scrubbing. Note the line starting with "To commit", which gives instructions on how to commit the change made by migrate.
```
$ Moe migrate --config_file moe_config.txt --db moe_db.txt --name test
Reading config file from moe_config.txt... Done
Performing migration 'test'... 
    Evaluating Codebase "internal>public"... 
        Creating from "internal"... /tmp/hg_archive_internal_4c0c54c3045deea0ea0d94fc13969734b7642b3d_1483780837009036036
        Translating /tmp/hg_archive_internal_4c0c54c3045deea0ea0d94fc13969734b7642b3d_1483780837009036036 from project space "internal" to "public"... 
            Editing /tmp/hg_archive_internal_4c0c54c3045deea0ea0d94fc13969734b7642b3d_1483780837009036036 with editor "scrub_step"... /tmp/expanded_tar_494920102821593557
        DONE: Translating /tmp/hg_archive_internal_4c0c54c3045deea0ea0d94fc13969734b7642b3d_1483780837009036036 from project space "internal" to "public": /tmp/expanded_tar_494920102821593557
    DONE: Evaluating Codebase "internal>public": /tmp/expanded_tar_494920102821593557
    Creating Writer "googlecode"... /tmp/hg_clone_googlecode_4614611359151548813
    Putting files from Codebase into Writer... 
        To commit, run: cd /tmp/hg_clone_googlecode_4614611359151548813 && ./hg_commit.sh && cd -
    DONE: Putting files from Codebase into Writer: Done
DONE: Performing migration 'test': Done
Created Draft Revisions:
/tmp/hg_clone_googlecode_4614611359151548813 in repository googlecode
```
The one or more --name flags are optional; when omitted, migrate performs all migrations listed in the config.