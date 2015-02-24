#!/bin/bash

# source the ciop functions (e.g. ciop-log)
source ${ciop_job_include}

export BEAM_HOME=${_CIOP_APPLICATION_PATH}/share/beam-4.11
export PATH=${BEAM_HOME}/bin:${PATH}

# define the exit codes
SUCCESS=0
ERR_NOINPUT=1
ERR_BINNING=2
ERR_NOPARAMS=5
ERR_JPEGTMP=7
ERR_BROWSE=9
ERR_INPUT_TAR=10
ERR_INPUT_COPY=11
ERR_NO_PUBLISH=12
ERR_PCONVERT=13

# add a trap to exit gracefully
function cleanExit ()
{
    local retval=$?
    local msg=""

    case "${retval}" in
        ${SUCCESS})        msg="Processing successfully concluded";;
        ${ERR_NOPARAMS})   msg="Output format not defined";;
        ${ERR_GDAL})       msg="Graph processing of job ${JOBNAME} failed (exit code ${res})";;
        ${ERR_NO_PUBLISH}) msg="Error while publishing results";;
        ${ERR_INPUT_COPY}) msg="Error while retrieving input";;
        ${ERR_INPUT_TAR})  msg="Error while untarring input";;
        ${ERR_PCONVERT})   msg="Error while executing pconvert";;
        ${ERR_JPEGTMP})    msg="Error while creating jpeg";;
        ${ERR_BROWSE})     msg="Error while creating browse";;
        *)                 msg="Unknown error";;
    esac
   
    [ ${retval} -ne 0 ] && ciop-log "ERROR" "Error ${retval} - ${msg}, processing aborted" || ciop-log "INFO" "${msg}"
    exit ${retval}
}
trap cleanExit EXIT

function get_xml_val() {
    cat $1 | grep $2 | cut -d '>' -f 2 | cut -d '<' -f 1 
}

function get_text_val() {
    cat $1 | grep $2 | cut -d '"' -f 2 | cut -d '"' -f 1
}

function get_data() {
    local url
    url="$1"

    prod=$( echo ${url} | ciop-copy -U -o ${TMPDIR}/input - )
    # checking if the copy was successfull
    [ $? -eq 0 ] && [ -n "${prod}" ] || return ${ERR_INPUT_COPY}
    
    cd ${TMPDIR}/input
    tar xfz $( basename ${prod} ) &> /dev/null

    # let's check the return value
    [ $? -eq 0 ] || return ${ERR_INPUT_TAR}
    cd -
}

function create_request_xml(){
    local l3db
    local request_xml_file

    l3db="$1"
    request_xml_file="$2"

    # retrieve the parameters value from workflow or job default value
    cellsize="$( ciop-getparam cellsize )"
    bandname="$( ciop-getparam bandname )"
    bitmask="$( ciop-getparam bitmask )"
    bbox="$( ciop-getparam bbox )"
    algorithm="$( ciop-getparam algorithm )"
    outputname="$( ciop-getparam outputname )"
    compress="$( ciop-getparam compress )"
    band="$( ciop-getparam band )"
    tailor="$( ciop-getparam tailor )"

    xmin=$( echo ${bbox} | cut -d "," -f 1 )
    ymin=$( echo ${bbox} | cut -d "," -f 2 )
    xmax=$( echo ${bbox} | cut -d "," -f 3 )
    ymax=$( echo ${bbox} | cut -d "," -f 4 )

    # first part of request file
    cat > ${request_xml_file} << EOF
<?xml version="1.0" encoding="ISO-8859-1"?>
  <RequestList>
    <Request type="BINNING">
      <Parameter name="process_type" value="init" />
      <Parameter name="database" value="${l3db}" />
      <Parameter name="lat_min" value="${ymin}" />
      <Parameter name="lat_max" value="${ymax}" />
      <Parameter name="lon_min" value="${xmin}" />
      <Parameter name="lon_max" value="${xmax}" />
      <Parameter name="log_prefix" value="l3" />
      <Parameter name="log_to_output" value="false" />
      <Parameter name="resampling_type" value="binning" />
      <Parameter name="grid_cell_size" value="${cellsize}" />
      <Parameter name="band_name.0" value="${bandname}" />
      <Parameter name="bitmask.0" value="${bitmask}" />
      <Parameter name="binning_algorithm.0" value="${algorithm}" />
      <Parameter name="weight_coefficient.0" value="1" />
    </Request>
    <Request type="BINNING">
      <Parameter name="process_type" value="update" />
      <Parameter name="database" value="${l3db}" />
      <Parameter name="log_prefix" value="l3" />
      <Parameter name="log_to_output" value="false" />
EOF

    for myfile in $( find ${TMPDIR}/input -type f -name "*.dim" )
    do
        echo "      <InputProduct URL=\"file://${myfile}\" /> " >> ${request_xml_file}
    done

    cat >> ${request_xml_file} << EOF
    </Request>
    <Request type="BINNING">
      <Parameter name="process_type" value="finalize" />
      <Parameter name="database" value="${l3db}" />
      <Parameter name="delete_db" value="true" />
      <Parameter name="log_prefix" value="l3" />
      <Parameter name="log_to_output" value="false" />
      <Parameter name="tailor" value="${tailor}" />
      <OutputProduct URL="file:${TMPDIR}/output/${outputname}.dim" format="BEAM-DIMAP" />
    </Request>
</RequestList>
EOF
}

function main(){
    # main function

    l3db_file=${TMPDIR}/l3_database.bindb
    request_xml_file=${TMPDIR}/binning_request.xml

    # creates the request xml
    create_request_xml "${l3db_file}" "${request_xml_file}"

    # starting binning
    binning.sh ${request_xml_file}
    [ $? -eq 0 ] || return ${ERR_BINNING}

    ciop-log "INFO" "Publishing binned DIMAP product"
    ciop-publish -m ${TMPDIR}/output/${outputname}.dim
    [ $? -eq 0 ] || return ${ERR_NO_PUBLISH}

    ciop-publish -m ${TMPDIR}/output/${outputname}.data
    [ $? -eq 0 ] || return ${ERR_NO_PUBLISH}

    cat > ${TMPDIR}/palette.cpd << EOF
$( ciop-getparam palette )
EOF

    ciop-log "INFO" "Generating image files"
    pconvert.sh -f png -b ${band} ${TMPDIR}/output/${outputname}.dim -c ${TMPDIR}/palette.cpd -o ${TMPDIR}/output &> /dev/null
    [ $? -eq 0 ] || return ${ERR_PCONVERT}

    ciop-publish -m ${TMPDIR}/output/${outputname}.png &> /dev/null
    [ $? -eq 0 ] || return ${ERR_NO_PUBLISH}
 
    pconvert.sh -f tif -b ${band} ${TMPDIR}/output/${outputname}.dim -c  ${TMPDIR}/palette.cpd -o ${TMPDIR}/output &> /dev/null
    [ $? -eq 0 ] || return ${ERR_PCONVERT}
    mv ${TMPDIR}/output/${outputname}.tif ${TMPDIR}/output/${outputname}.rgb.tif
    ciop-publish -m ${TMPDIR}/output/${outputname}.rgb.tif
    [ $? -eq 0 ] || return ${ERR_NO_PUBLISH}
  
    pconvert.sh -f tif -b ${band} ${TMPDIR}/output/${outputname}.dim -o ${TMPDIR}/output &> /dev/null
    [ $? -eq 0 ] || return ${ERR_PCONVERT}
    ciop-publish -m ${TMPDIR}/output/${outputname}.tif
    [ $? -eq 0 ] || return ${ERR_NO_PUBLISH}

    dim=${TMPDIR}/output/${outputname}.dim

    width=$( get_xml_val ${dim} NCOLS )
    height=$( get_xml_val ${dim} NROWS )
    minx=$( get_text_val ${dim} EASTING )
    maxy=$( get_text_val ${dim} NORTHING )
    resx=$( get_text_val ${dim} PIXELSIZE_X )
    resy=$( get_text_val ${dim} PIXELSIZE_Y )
    maxx=$( echo "$minx + $width * $resx" | bc -l )
    miny=$( echo "$maxy - $height * $resy" | bc -l )

    convert -cache 1024 -size ${width}x${height} -depth 8 -interlace Partition ${TMPDIR}/output/${outputname}.png ${TMPDIR}/tmp.jpeg &> /dev/null
    [ $? -eq 0 ] || return ${ERR_JPEGTMP}
  
    ciop-log "INFO" "Generating the browse"
    convert -cache 1024 -size 150x150 -depth 8 -interlace Partition ${TMPDIR}/tmp.jpeg ${TMPDIR}/output/${outputname}_browse.jpg &> /dev/null
    [ $? -eq 0 ] || return ${ERR_BROWSE}
    ciop-publish -m ${TMPDIR}/output/${outputname}_browse.jpg
    [ $? -eq 0 ] || return ${ERR_NO_PUBLISH}

    return ${SUCCESS}
}

mkdir -p $TMPDIR/input
mkdir -p $TMPDIR/output

while read product
do
    get_data "${product}"
done

main || exit $?
