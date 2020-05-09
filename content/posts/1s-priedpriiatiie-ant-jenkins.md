---
title: "1С:Предприятие + Ant + jenkins"
date: 2013-03-16
draft: false
---

1С:Предприятие обладает очень скудными средствами разработки и интеграции. Однако выжать максимум автоматизации сборки и публикации можно.
Мои проекты имеют следующий жизненный цикл:

1. Внесение изменений в код согласно `Jira`
2. Выпуск тестовой сборки
3. Исправление ошибок по необходимости
4. Выпуск релиза (проверка конфигурации, формирование `cf` + `cfu`, формирование отчета по измененным объектам хранилища конфигурации)
5. Изменение версии в хранилище конфигурации

Для выпуска релиза мной разработан `build.xml` для `Apache Ant`

``` xml
<project name="kisnpo" default="build">
    <description>kisnpo build</description>
    <property environment="env"/>
    <property name="rep" value="tcp://share\myproject\"/>
    <property name="replogin" value="login"/>
    <property name="reppassword" value="password"/>
    <property name="ibname" value="c:\ib\myproject"/>
    <property name="ib" value="/F${ibname}"/>
    <property name="iblogin" value="login"/>
    <property name="ibpassword" value="password"/>
    <property name="connectionstring" value="Usr="${iblogin}";Pwd="${ibpassword}";File="${ibname}";"/>
    <property name="1cbin" location="C:\Program Files (x86)\1cv81\bin\1cv8.exe"/>
    <property name="1cbincom" value="V81.COMConnector"/>
    <property name="builds" location="${env.WORKSPACE}"/>
    <property name="publishdebug" location="\\ftp\project-test"/>
    <property name="publish" location="\\ftp\project"/>
    <property name="NBegin" value="1"/>
    <property name="NEnd" value="1"/>
    <target name="init" depends="build-version">
        <fail if="version_present" message="Version ${version} is builed"/>
        <echo message="ok"/>
    </target>
    <target name="build-version" unless="version">
        <exec executable="powershell.exe" output=".version" failonerror="true">
            <arg value="$c=new-object -comobject \"${1cbincom}\";"/>
            <arg value="$b=$c.Connect(\"${connectionstring}\");"/>
            <arg value="$medata=[System.__ComObject].invokemember(\"Metadata\",[System.Reflection.BindingFlags]::GetProperty,$null,$b,$null);"/>
            <arg value="[String]$buildversion = [System.__ComObject].invokemember(\"Version\",[System.Reflection.BindingFlags]::GetProperty,$null,$medata ,$null);"/>
            <arg value="echo $buildversion;"/>
            <arg value="$rb=[System.Runtime.Interopservices.Marshal]::ReleaseComObject($b);"/>
            <arg value="$rc=[System.Runtime.Interopservices.Marshal]::ReleaseComObject($c)"/>
        </exec>
        <loadfile property="version" srcFile=".version">
            <filterchain>
                <striplinebreaks/>
            </filterchain>
        </loadfile>
        <available filepath="${builds}" file="${version}" type="dir" property="version_present"/>
        <echo message="ok"/>
    </target>
    <target name="clean">
        <delete file=".version"/>
        <delete file=".checkerror"/>
        <delete file=".updatescf"/>
        <delete file=".resultcheck"/>
        <delete file=".report"/>
        <echo message="ok"/>
    </target>
    <target name="update">
        <exec executable="${1cbin}" failonerror="true">
            <arg value="CONFIG"/>
            <arg value="${ib}"/>
            <arg value="/N${iblogin}"/>
            <arg value="/P${ibpassword}"/>
            <arg value="/ConfigurationRepositoryF${rep}"/>
            <arg value="/ConfigurationRepositoryN${replogin}"/>
            <arg value="/ConfigurationRepositoryP${reppassword}"/>
            <arg value="/ConfigurationRepositoryUpdateCfg"/>
            <arg value="-force /UpdateDBCfg"/>
        </exec>
        <echo message="ok"/>
    </target>
	<target name="check" depends="update">
		<exec executable="${1cbin}" output=".resultcheck" failonerror="true">
			<arg value="CONFIG"/>
			<arg value="${ib}"/>
			<arg value="/N${iblogin}"/>
			<arg value="/P${ibpassword}"/>
			<arg value="/ConfigurationRepositoryF${rep}"/>
			<arg value="/ConfigurationRepositoryN${replogin}"/>
			<arg value="/ConfigurationRepositoryP${reppassword}"/>
			<arg value="/CheckConfig"/>
			<arg value="-IncorrectReferences"/>
			<arg value="-Client"/>
			<arg value="-ClientServer"/>
			<arg value="-ExternalConnectionServer"/>
			<arg value="-ExternalConnection"/>
			<arg value="-Server"/>
			<arg value="-DistributiveModules"/>
			<arg value="-ConfigLogicalIntegrity"/>
			<arg value="/DumpResult .checkerror"/>
		</exec>
		<loadfile property="checkerror" srcFile=".checkerror" encoding="utf-8">
			<filterchain>
				<tokenfilter>
					<containsregex pattern="^[^0-9\-]+(0)$"/>
				</tokenfilter>
			</filterchain>
		</loadfile>
		<fail unless="checkerror"/>
		<echo message="ok"/>
	</target>
	<target name="build" depends="check,init" unless="version_present">
		<exec executable="cmd.exe">
			<arg value="/c"/>
			<arg value="rd"/>
			<arg value=".last"/>
		</exec>
		<echo message="Build version is ${version}"/>
		<exec executable="powershell.exe" output=".updatescf" failonerror="true">
			<arg value="[String]$p=\"\";"/>
			<arg value="ls ${builds}\* -Include 1cv8.cf -Recurse | split-path -parent | split-path -leaf | foreach {$p = $p + \" -f $_\1cv8.cf -v $_ \"};"/>
			<arg value="echo $p"/>
		</exec>
		<loadfile property="updatescf" srcFile=".updatescf">
			<filterchain>
				<striplinebreaks/>
			</filterchain>
		</loadfile>
		<exec executable="${1cbin}" output=".resultcheck" failonerror="true">
			<arg value="CONFIG"/>
			<arg value="${ib}"/>
			<arg value="/N${iblogin}"/>
			<arg value="/P${ibpassword}"/>
			<arg value="/ConfigurationRepositoryF${rep}"/>
			<arg value="/ConfigurationRepositoryN${replogin}"/>
			<arg value="/ConfigurationRepositoryP${reppassword}"/>
			<arg value="/CreateDistributionFiles -cffile ${version}\1cv8.cf -cfufile ${version}\1cv8.cfu"/>
			<arg value="${updatessubcf}"/>
			<arg value="${updatescf}"/>
		</exec>
		<sleep seconds="30"/>
		<exec executable="cmd.exe">
			<arg value="/c"/>
			<arg value="mklink"/>
			<arg value="/J"/>
			<arg value=".last"/>
			<arg value="${version}"/>
		</exec>
		<echo message="ok"/>
	</target>
	<target name="publishdebug" depends="build-version" if="${publishdebugenable}">
		<copy todir="${publishdebug}/${version}">
			<fileset dir="${builds}/${version}"/>
		</copy>
		<echo message="ok"/>
	</target>
	<target name="publish" depends="build-version" if="${publishenable}">
		<copy todir="${publish}/${version}">
			<fileset dir="${builds}/${version}"/>
		</copy>
		<echo message="ok"/>
	</target>
	<target name="report">
		<exec executable="${1cbin}">
			<arg value="CONFIG"/>
			<arg value="${ib}"/>
			<arg value="/N${iblogin}"/>
			<arg value="/P${ibpassword}"/>
			<arg value="/ConfigurationRepositoryF${rep}"/>
			<arg value="/ConfigurationRepositoryN${replogin}"/>
			<arg value="/ConfigurationRepositoryP${reppassword}"/>
			<arg value="/ConfigurationRepositoryReport ${version}/.report -NBegin ${NBegin} -NEnd ${NEnd} -GroupByObject -GroupByComment"/>
		</exec>
		<sleep seconds="30"/>
	</target>
</project>
```

Данный build.xml подразумевает следующую структуру каталогов:

```none
<Builds>
    [Project name]
        [.last]      - ссылка на последнюю мажорную версию
        [Version 1]
        [Version 2]
        ...
        [Version n]
            [.last]  - ссылка на последнюю минорную версию
            [Buld 1]
            [Buld 2]
            ...
            [Buld m]
```

Например:

``` none
MyProject
    [.last]
    [1.0.1]
        [.last]
        [1.0.1.1]
        [1.0.1.2]
        [1.0.1.3]
        [1.0.1.4]
    [1.0.2]
        [.last]
        [1.0.1.4] - ссылка на предыдущую последнюю (релизную) мажорную сборку
        [1.0.2.1]
        [1.0.2.2]
```

Для каждой мажорной версии в `jenkins` создается своя задача. `CI`-системы Bamboo или `Team City` позволяют добиться более удобного представления проектов, но пока мне хватает простоты и бесплатности `Jenkins`.

Переключение версии никакими стандартными средствами `1С` не предусмотрено, так же никак нельзя автоматизированно узнать номера изменений хранилища конфигурации для построения отчета по измененным объектам.

В чем плюсы такой связки? Во-первых, это упрощает и автоматизирует типовые этапы жизненного цикла проектов. Да - можно было написать все просто скриптами (как и было до этого), но в конечном счете этот путь приведет к написанию чего-нибудь наподобие `Apache Ant`, когда нужно на этапе выполнения определить зависимость целей и все необходимые параметры. Во-вторых, используя `Apache Ant` мы обеспечиваем возможность работы с различными современными `CI`.