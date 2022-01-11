# YarnParser
dogshit yarn v1 mappings parser

tested on java 17 and [these](https://maven.fabricmc.net/net/fabricmc/yarn/) mappings ([example](https://maven.fabricmc.net/net/fabricmc/yarn/1.18.1%2Bbuild.18/yarn-1.18.1%2Bbuild.18-tiny.gz))

# Example
```
package fuck.you.yarnparser;

import fuck.you.yarnparser.entry.ClassEntry;
import fuck.you.yarnparser.entry.FieldEntry;
import fuck.you.yarnparser.entry.MethodEntry;

import java.io.File;
import java.util.List;

public class Test
{
    public static void main( String[ ] args )
    {
        System.out.println( "hi" );

        File mappings = new File( "D:\\yarn-1.18.1+build.18-tiny" );

        try
        {
            long start = System.currentTimeMillis( );

            // you can pass an InputStream instead
            V1Parser parser = new V1Parser( mappings );

            long end = System.currentTimeMillis( );

            System.out.println( "Parsed in " + ( end - start ) + "ms" );

            List< ClassEntry > classes = parser.getClasses( );
            List< FieldEntry > fields = parser.getFields( );
            List< MethodEntry > methods = parser.getMethods( );

            System.out.println( String.format( "%d classes, %d fields, %d methods",
                    classes.size( ), fields.size( ), methods.size( ) ) );

            System.out.println( );
            System.out.println( "Classes:" );
            for( ClassEntry entry : classes )
            {
                System.out.println( String.format( "-> [class] official: %s | intermediary: %s | named: %s",
                        entry.official, entry.intermediary, entry.named ) );
            }

            System.out.println( );
            System.out.println( "Fields:" );
            for( FieldEntry entry : fields )
            {
                System.out.println( String.format( "-> [field] class: %s | intermediary: %s | named: %s",
                        entry.official, entry.intermediary, entry.named ) );
            }

            System.out.println( );
            System.out.println( "Methods:" );
            for( MethodEntry entry : methods )
            {
                System.out.println( String.format( "-> [method] class: %s | intermediary: %s | named: %s",
                        entry.official, entry.intermediary, entry.named ) );
            }

            // you can pass a null to classname argument and it wont check the class
            // good for finding intermediary mappings (field_8235 etc..)
            // but bad for finding named mappings (player, world, stepHeight, netHandler etc...)
            testIntermediaryField( parser, null, "field_1724", "player" );
            testIntermediaryField( parser, null, "field_6013", "stepHeight" );
            testNamedField( parser, "net/minecraft/class_310", "player", "field_1724" );
            testNamedField( parser, "net/minecraft/class_1297", "stepHeight", "field_6013" );
        }
        catch( Exception e )
        {
            e.printStackTrace( );
        }
    }

    public static void testIntermediaryField( V1Parser parser, String classname, String intermediary, String expectation )
    {
        System.out.println( );
        System.out.println( String.format( "[testIntermediaryField] Trying to find %s (expecting %s)", intermediary, expectation ) );

        FieldEntry field = parser.findField( classname, intermediary, V1Parser.NormalFindType.INTERMEDIARY );
        if( field == null )
        {
            System.out.println( "[testIntermediaryField] [ERROR] Failed to find " + intermediary );
            return;
        }

        System.out.println( String.format( "[testIntermediaryField] -> [field] class: %s | intermediary: %s | named: %s",
                field.official, field.intermediary, field.named ) );

        if( !field.named.equals( expectation ) )
        {
            System.out.println( String.format( "[testIntermediaryField] [ERROR] Found %s but got \"%s\" yarn mapping (expected \"%s\")",
                    intermediary, field.named, expectation ) );
        }
    }

    public static void testNamedField( V1Parser parser, String classname, String named, String expectation )
    {
        System.out.println( );
        System.out.println( String.format( "[testNamedField] Trying to find %s (expecting %s)", named, expectation ) );

        FieldEntry field = parser.findField( classname, named, V1Parser.NormalFindType.NAMED );
        if( field == null )
        {
            System.out.println( "[testNamedField] [ERROR] Failed to find " + named );
            return;
        }

        System.out.println( String.format( "[testNamedField] -> [field] class: %s | intermediary: %s | named: %s",
                field.official, field.intermediary, field.named ) );

        if( !field.intermediary.equals( expectation ) )
        {
            System.out.println( String.format( "[testNamedField] [ERROR] Found %s but got \"%s\" (expected \"%s\")",
                    named, field.named, expectation ) );
        }
    }
}
```

[output](https://gist.github.com/mr-nv/fc69e5223d605961a7d518746587f269)
