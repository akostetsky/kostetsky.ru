# Blog kostetsky.ru #

## Setup ##

```sh
git clone git@bitbucket.org:akostetskiy/kostetsky.ru.git
cd kostetsky.ru
conda env create -f environment.yml
conda activate pelican
```

## Env activate ##
```sh
conda activate pelican
```

## Automatically reload browser tab upon file modification. 
```sh
invoke livereload
```    

## Upgrading ##
```sh
conda env update --name=pelican --file=environment.yml
```

## Build ##
```sh
invoke build
```

## Publish ##
```sh
invoke publish
```