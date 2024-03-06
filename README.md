# Initial Config

## Download galasactl
galasactl is available on github https://github.com/galasa-dev/cli/releases for the following platforms:
* MacOS (both Intel and ARM architecture)
* Windows
* Linux
* z/OS

## Create a project
`galasactl project create --package sample.project --obr --maven --log -`

## Run a test
`galasactl runs submit local --obr mvn:sample.project/sample.project.obr/0.0.1-SNAPSHOT/obr --class sample.project.feature1/sample.project.feature1.TestFeature1 --log -`

# Connect to 3270
## pom updates
		<dependency>
			<groupId>dev.galasa</groupId>
			<artifactId>dev.galasa.zos3270.manager</artifactId>
			<scope>provided</scope>
		</dependency>

## Test Objects
`@Zos3270Terminal(imageTag = "REGION")
	public ITerminal terminal;`

## CPS Updates
```
# Plex2
zos.cluster.PLEX2.images=MV2D

zos.image.MV2D.default.hostname=winmvs2d.hursley.ibm.com
zos.image.MV2D.ipv4.hostname=winmvs2d.hursley.ibm.com
zos.image.MV2D.sysplex=PLEX2
zos.image.MV2D.telnet.port=23
zos.image.MV2D.telnet.tls=false

cicsts.provision.type=DSE
cicsts.dse.tag.REGION.applid=IYK2ZNB5
cicsts.dse.tag.REGION.imageid=MV2D
cicsts.default.logon.initial.text=HIT ENTER FOR LATEST STATUS
cicsts.default.logon.gm.text=******\\(R)
```
## Test code
```
@Test
	public void mainframeTest() throws Exception{
		terminal.waitForTextInField("HIT ENTER FOR LATEST STATUS");
		terminal.reportScreen();

		terminal.type("logon applid(IYK2ZNB5)").enter().waitForTextInField("CICS FOR GALASA TEST");
		terminal.waitForKeyboard().clear().waitForKeyboard();
		terminal.type("TSQT").enter();

		assertThat(terminal.searchText("HELLO WORLD")).isTrue();
	}
```

# Adding CICS

## pom updates
```
<dependency>
			<groupId>dev.galasa</groupId>
			<artifactId>dev.galasa.cicsts.manager</artifactId>
			<scope>provided</scope>
		</dependency>
        
```

## CPS updates
```
cicsts.provision.type=DSE
cicsts.dse.tag.REGION.applid=IYK2ZNB5
cicsts.dse.tag.REGION.imageid=MV2D
cicsts.default.logon.initial.text=HIT ENTER FOR LATEST STATUS
cicsts.default.logon.gm.text=******\\(R)
```

# New Test objects
```
@CicsRegion(cicsTag = "REGION")
	public ICicsRegion region;

	@CicsTerminal(cicsTag = "REGION")
	public ICicsTerminal cicsTerminal;
```

#Test updates
```
@Test
	public void mainframeTest() throws Exception{
		cicsTerminal.type("TSQT").enter();

		assertThat(cicsTerminal.searchText("HELLO WORLD")).isTrue();
	}
```

```
@Test
	public void mainframeTest() throws Exception{
		cicsTerminal.type("TSQT").enter();

		assertThat(cicsTerminal.searchText("HELLO WORLD")).isTrue();
		cicsTerminal.clear().wfk().type("CECI").enter().waitForKeyboard();

		region.ceci().issueCommand(cicsTerminal, "CECI READQ TS QUEUE('GALASA') INTO(&DATA)");
		String response = region.ceci().retrieveVariableText(cicsTerminal, "DATA");
		assertThat(response).contains("HELLO WORLD");
	}
```

