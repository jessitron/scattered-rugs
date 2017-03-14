/*
 * Copyright © 2017 Atomist, Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

import { EditProject } from '@atomist/rug/operations/ProjectEditor';
import { Project } from '@atomist/rug/model/Project';
import { Pattern } from '@atomist/rug/operations/RugOperation';
import { Editor, Parameter, Tags } from '@atomist/rug/operations/Decorators';
import { PathExpression, PathExpressionEngine, Microgrammar, TextTreeNode } from '@atomist/rug/tree/PathExpression';
import { File } from '@atomist/rug/model/File';
import { Or } from '@atomist/rug/tree/Microgrammars'

@Editor("ClojureFunctionParametersToMap", "do this refactor I want right now")
@Tags("clojure")
export class ClojureFunctionParametersToMap implements EditProject {

    edit(project: Project) {

        let functionOfInterest = "register-message-handler"

        let mg = new Microgrammar("functionCall", `($fnName $arg $arg $arg $arg $arg $arg $arg $arg $arg)`,
            {
                arg: Or([`§"[^"]*"§`, `#§"[^"]*"§`, `§[:a-zA-Z-]*§`, `(constantly true)`]),
                namespace: `§[#'\.a-z-]*§`,
                fnName: Or([`$namespace/${functionOfInterest}`, functionOfInterest])
            })

        let mg2 = new Microgrammar("functionDeclaration",
            `(defn ${functionOfInterest} $docstring ⟦[⟧$param $param $param $param $param $param $param $param $param⟦]⟧`
            , {
                param: `§[a-zA-Z-]+§`,
                docstring: `"§[^"]*§"`
            })

        const eng: PathExpressionEngine = project.context().pathExpressionEngine().addType(mg).addType(mg2);


        let mapKeys: string[];
        eng.with<any>(project.findFile('src/atomist/bot/bot.clj'), `/functionDeclaration()`, fn => {
            let paramNames: string[] = fn.param().map(p => { return p.value() });

            mapKeys = paramNames.map(p => { return ":" + p });
            console.log(`map keys: ${mapKeys}`)
            fn.update(
                `(defn ${functionOfInterest}
    ${fn.docstring().value()}
    [${paramNames[0]} {:keys [${paramNames.slice(1).join(" ")}]}]`)
        })
        if (mapKeys === undefined) {
            console.log("FAILURE: Function declaration not identified")
        } else {
            // how do I throw an exception?


            eng.with<File>(project, `//File()`, e => {
                if (e.path().indexOf(".clj") > 0) {
                    //console.log(`checking file: ${e.path()}`)
                    eng.with<any>(e, `/functionCall()`, f => {
                        console.log(`Found an example in ${e.path()}: ${f.value()}`)

                        let children = f.arg().map(c => { return c.value(); });
                        console.log(`   first arg: ${children[0]}, children size ${children.length}`)

                        let mapEntries = this.zip(children, mapKeys, (c, k) => { return `${k} ${c}` })
                        console.log(`Map entries: ${mapEntries}`)

                        // I think there's a way to get the indentation right
                        let newCall = `(${f.fnName().value()} ${children[0]} {${mapEntries.slice(1).join("\n                                        ")} } )`;

                        console.log(`new value: ${newCall}`);

                        f.update(newCall);
                    })
                }
            });
        }
    }

    zip<T, S>(ar1: Array<T>, ar2: Array<T>, f: (t1: T, t2: T) => S) {
        const length = Math.min(ar1.length, ar2.length);
        let out: S[] = [];
        for (let i = 0; i < length; i++) {
            out.push(f(ar1[i], ar2[i]));
        }
        return out;
    }
}

export const editorClojureFunctionParametersToMap = new ClojureFunctionParametersToMap()
