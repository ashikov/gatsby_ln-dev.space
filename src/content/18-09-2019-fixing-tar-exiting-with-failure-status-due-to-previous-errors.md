---
title: "Ошибка tar: Exiting with failure status due to previous errors"
date: "2019-09-18"
draft: false
path: "/blog/18-09-2019-fixing-tar-exiting-with-failure-status-due-to-previous-errors"
---

>Люди постоянно спрашивают у меня на улицах следующее: "Слушай, иногда, когда я архивирую большую важную директорию [tar-ом](https://en.wikipedia.org/wiki/Tar_(computing)) процесс обрывается с ошибкой - __tar: Exiting with failure status due to previous errors__. При этом, если пробежаться глазами по выводу в терминал, то никаких ошибок там нет. В чем может быть проблема?".

Давайте разбираться.

Обычно данная ошибка связана с отсутствием прав для доступа к файлу который нужно упаковать в архив. Но тогда почему [tar](https://en.wikipedia.org/wiki/Tar_(computing)) даже в визуальном (__-v verbose mode__) режиме не указывает на этот файл?

Дело в том, что информация об ошибке все-таки выводится, но её легко пропустить, так как она попадает в поток [STDERR](https://en.wikipedia.org/wiki/Standard_streams#Standard_error_(stderr)).

Для того, чтобы отловить строки с указанием причины возникновения ошибки, нужно отфильтровать поток [STDOUT](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout)). Просто отправим его в [/dev/null](https://en.wikipedia.org/wiki/Null_device). :)

```shell
ln@dev:~$ tar cvfz archive.tgz directory/ > /dev/null
```

Увидим примерно следующее:

```shell
tar: directory/file.db.~lock~: Cannot open: Permission denied
tar: Exiting with failure status due to previous errors
```

Осталось разобраться с правами доступа к файлу или просто удалить его, если он вам не нужен и продолжить заниматься упаковкой архива, потягивая из любимой кружки только, что заваренный крупнолистовой цейлонский черный чай.
